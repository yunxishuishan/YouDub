---
name: dev-tips
description: 编程开发技巧合集。在写代码、改 bug、重构、提交、评审、讨论工程实践，或用户要求把新技巧写入本 skill 时使用。覆盖：如何按统一模板追加技巧；C++ 语法（命名空间避免重名）；Meson 的 meson.build（声明项目、依赖、目标和子目录）。
---

# 开发技巧集

Agent 在相关任务中应遵循本文件中的约定。条目按主题分章；用户后续给出的技巧插入对应章节，不要另建 skill 文件。

## 用法

- 优先按当前任务匹配章节，不要整份照搬无关条目。
- 技巧与仓库现有代码冲突时，以本文件明示约定为准，并在回复里说明依据。
- 用户补充新技巧时，按 [维护本技巧集](#1-维护本技巧集) 追加，并同步更新文首 `description` 与下方目录。

## 目录

- [1. 维护本技巧集](#1-维护本技巧集)
- [2. C++ 语法](#2-c-语法)
- [3. Meson 构建](#3-meson-构建)

## 1. 维护本技巧集

### 按统一模板追加技巧

**适用：** 用户口述、贴命令、贴代码，或说「遇到 X 要 Y」，要求记入本 skill。

**做法：**

1. 起一个短标题，归入现有章节；没有合适章节则新建一章，并更新本文件「目录」。
2. 把内容压成可执行约定，使用下面的条目结构（无反例或示例时可省略对应小节）：

   ```markdown
   ### 技巧标题

   **适用：** …
   **做法：** …
   **避免：** …
   **示例：**
   ```

3. 视内容微调 YAML `description`：补上新覆盖的主题词，方便自动触发。
4. 相关条目互相引用，避免重复段落。
5. 只改本文件（[`.cursor/skills/dev-tips/SKILL.md`](.cursor/skills/dev-tips/SKILL.md)），不拆成多个 skill。

**避免：** 写成散文笔记；不要把无关业务代码改动塞进本文件。

## 2. C++ 语法

### 命名空间是为了避免重名

**适用：** 写或改 C++ 时，多个库、模块或文件可能声明同名类型、函数、常量或变量。

**做法：**

- 用 `namespace` 把标识符圈进独立作用域，靠限定名（`ns::name`）或有节制的 `using` 区分来源。
- 新增对外符号时优先放进本模块/本库的命名空间，而不是默认丢进全局命名空间。
- 头文件里避免 `using namespace ...;`，以免把名字再次泄漏到包含方，抵消隔离效果。

**避免：** 为了少打字而在头文件或大范围里 `using namespace`，导致重名冲突重新出现。

**示例：**

```cpp
namespace audio {
void mix();
}

namespace video {
void mix();
}

audio::mix();
video::mix();
```

## 3. Meson 构建

### meson.build 的作用

**适用：** 读、改或排查 C/C++（及其他 Meson 项目）的构建；文件名是 `meson.build`（子目录里也可以各有一份）。

**做法：** 把它当成 Meson 的构建说明书（地位类似 CMake 的 `CMakeLists.txt`），只描述「编什么」，真正的编译命令由 `meson setup` 生成到 build 目录（常见是 `build.ninja`）。一份 `meson.build` 主要做这些事：

| 职责 | 常见写法 | 说明 |
| --- | --- | --- |
| 声明工程 | `project('name', 'cpp', version: '…')` | 根目录那一份必须有；语言、版本、默认 option 也写这里 |
| 找依赖 | `dependency('foo')` / `cc.find_library()` | pkg-config、CMake config、系统库等 |
| 定义产物 | `executable()` / `library()` / `shared_module()` | 源文件、头文件目录、编译/链接参数绑在目标上 |
| 拆分子目录 | `subdir('src')` | 每个子目录自己的 `meson.build` 只描述该目录，由上层引入 |
| 测试 | `test('name', exe)` | `meson test` 跑 |
| 安装 | `install: true` 或 `install_headers()` | `meson install` 用 |
| 读选项 | `get_option('foo')` | 选项本身定义在 `meson_options.txt`，不是 `meson.build` |

改构建时：先改对应的 `meson.build`，再在已有 build 目录里 `meson compile`（必要时 `meson setup --reconfigure`），不要手改生成出来的 `build.ninja`。

**避免：**

- 把 `meson.build` 当成 Makefile 去写具体编译器命令；Meson 脚本不是 shell。
- 把选项定义、wrap 依赖和构建描述混在一个文件里：选项看 `meson_options.txt`，第三方 wrap 看 `subprojects/`。
- 只改子目录的 `meson.build` 却忘了上层是否 `subdir()` 引入了它。

**示例：**

```meson
project('demo', 'cpp', version: '0.1', default_options: ['cpp_std=c++17'])

gtk = dependency('gtk+-3.0', required: false)

exe = executable(
  'demo',
  'src/main.cpp',
  dependencies: gtk,
  install: true,
)

test('smoke', exe)
subdir('po')
```
