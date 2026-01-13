---
name: qt-build-debug
description: 使用本地 Qt 工具链（qmake 或 CMake）编译、构建和调试 C++ 项目。当用户需要编译 Qt 项目、运行 qmake、执行 make 命令、配置调试会话或查找 Qt 相关构建问题时自动应用此 Skill。
---

# Qt 项目编译与调试助手

本 Skill 指导你使用本地 Qt 开发环境完成项目的配置、编译、构建和调试全流程。

## 核心指令

当你应用此 Skill 时，请遵循以下步骤来协助用户：

### 1. 项目识别与环境检查
- **Qt安装目录**：qt的安装目录在`D:\Qt5.15\5.15.2\msvc2019_64\bin`
- **确定构建系统**：首先检查项目根目录，识别是使用 `qmake`（存在 `.pro` 文件）还是 `CMake`（存在 `CMakeLists.txt` 文件）。
- **定位 Qt 安装**：尝试通过运行 `qmake --version` 或查看环境变量（如 `QTDIR`, `QT_VERSION`）来确认 Qt 的安装路径和版本。
- **检查依赖**：读取项目文件，识别可能缺失的 Qt 模块（如 `QT += core gui widgets`）或第三方库。

### 2. 项目配置与生成
- **对于文本编码**：
  - 确保项目文件使用 UTF-8 编码，避免因编码问题导致的编译错误。
  - 在WIndows下，要在头文件中增加 `#pragma execution_character_set("utf-8")` 来确保源文件以 UTF-8 编码编译。
- **对于 qmake 项目**：
  1. 在项目根目录下新建的 `build/<Qt版本号>_<Release或Debug>` 目录中运行 `qmake <path_to_project.pro>`。假如已经存在类似目录，可以直接进入该目录执行命令。
  2. 检查输出，确保没有出现 “Project ERROR” 之类的消息。
  3. 成功后，会生成 `Makefile`、`Makefile.Debug`、`Makefile.Release`。
- **对于 CMake 项目**：
  1. 建议创建一个独立的构建目录（如 `build/`）。
  2. 指导用户运行 `cmake .. -DCMAKE_PREFIX_PATH=<path_to_qt>` 来配置项目，其中 `<path_to_qt>` 是 Qt 的安装路径（例如 `/opt/Qt/5.15.2/gcc_64/lib/cmake`）。
  3. 检查 CMake 输出，确认找到了正确的 Qt 版本和所需组件。

### 3. 编译与构建
- 使用`D:\Qt5.15\Tools\QtCreator\bin\jom\jom.exe`进行编译。

### 4. 运行与测试
- 指导用户在构建目录下运行生成的可执行文件（如 `./your_qt_app`）。
- 由于dll等依赖问题，请将 Qt 的 `bin` 目录添加到运行时的环境变量 `PATH` 中，确保程序能找到所需的动态库。

### 4. 调试支持
- **启动调试器**：项目编译成功后，指导用户使用调试器。
  - **GDB/LLDB**：运行 `gdb ./your_qt_executable` 或 `lldb ./your_qt_executable`。
  - **Qt Creator**：如果用户使用 IDE，可以指导其如何导入项目并配置调试套件。
- **核心调试步骤**：
  1. 在代码中建议设置断点的关键位置（如 `main` 函数入口、信号槽连接处、疑似崩溃的函数）。
  2. 解释如何运行程序到断点（`run` 命令）。
  3. 解释基础调试命令：`next`（单步跳过）、`step`（单步进入）、`print`（查看变量）、`backtrace`（查看调用栈）。
  4. 对于 Qt 特定问题（如信号未触发、UI 不更新），指导如何检查事件循环、对象树和连接状态。

## 常用命令参考

将此作为快速命令手册，在指导用户时直接引用。

| 任务 | 命令示例 |
| :--- | :--- |
| **查询 Qt 版本** | `qmake --version` |
| **生成 Makefile (qmake)** | `qmake ../project.pro` 或 `qmake -spec linux-g++` |
| **CMake 配置** | `cmake .. -DCMAKE_PREFIX_PATH=/opt/Qt/6.5.0/gcc_64` |
| **编译项目** | `make` 或 `make -j4` |
| **清理构建** | `make clean` |
| **运行程序** | `./your_qt_app` |
| **GDB 调试** | `gdb --args ./your_qt_app arg1 arg2` |
| **查看 .pro 文件** | `cat yourproject.pro` |

## 故障排除指南

当用户遇到问题时，按此逻辑排查：
1.  **“qmake/CMake 未找到”**：检查 Qt 是否安装，环境变量 `PATH` 是否包含 Qt 的 `bin` 目录。
2.  **“头文件找不到”**：检查 `.pro` 文件中的 `INCLUDEPATH` 或 CMake 中的 `include_directories()`。
3.  **“库链接失败”**：检查 `.pro` 中的 `LIBS` 或 CMake 中的 `target_link_libraries()`，确保链接了正确的 Qt 模块（如 `-lQt5Core -lQt5Gui`）。
4.  **“程序启动崩溃”**：建议使用调试器获取堆栈跟踪。常见原因包括：插件路径不正确（设置 `QT_DEBUG_PLUGINS=1` 环境变量）、二进制与动态库版本不匹配。
5.  **“UI 文件未编译”**：确保 `.pro` 文件包含 `FORMS += mydialog.ui`，或 CMake 中使用了 `qt_wrap_ui()`。

始终以清晰、分步骤的方式向用户解释流程和命令，并在执行任何可能修改文件的命令前获得用户确认。