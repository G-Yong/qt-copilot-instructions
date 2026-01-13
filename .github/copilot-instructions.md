---
name: qt-build-debug
description: 使用本地 Qt 工具链（qmake 或 CMake）编译、构建和调试 C++ 项目。当用户需要编译 Qt 项目、运行 qmake、执行 make 命令、配置调试会话或查找 Qt 相关构建问题时自动应用此 Skill。
---

# Qt 项目编译与调试助手

本 Skill 指导你使用本地 Qt 开发环境完成项目的配置、编译、构建和调试全流程。

# 路径定义
这里定义一些路径，方便后面用变量来引用：
- **Qt 安装根目录**：`D:\Qt5.15\5.15.2\msvc2019_64`
- **Qt bin 目录**：`D:\Qt5.15\5.15.2\msvc2019_64\bin`
- **qmake 路径**：`D:\Qt5.15\5.15.2\msvc2019_64\bin\qmake.exe`
- **MSVC 环境脚本**：`C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvarsall.bat`  
- **jom 编译器路径**：`D:\Qt5.15\Tools\QtCreator\bin\jom\jom.exe`

## Windows PowerShell 环境说明
在 Windows PowerShell 环境下，由于需要初始化 MSVC 编译器环境，建议使用 `cmd /c` 来执行命令序列，确保环境变量在同一会话中生效。


## 核心指令

当你应用此 Skill 时，请遵循以下步骤来协助用户：

### 1. 项目识别与环境检查
- **qmake**：使用Qt安装目录下的 `qmake.exe`。
- **确定构建系统**：首先检查项目根目录，识别是使用 `qmake`（存在 `.pro` 文件）还是 `CMake`（存在 `CMakeLists.txt` 文件）。
- **定位 Qt 安装**：尝试通过运行 `qmake --version` 或查看环境变量（如 `QTDIR`, `QT_VERSION`）来确认 Qt 的安装路径和版本。
- **检查依赖**：读取项目文件，识别可能缺失的 Qt 模块（如 `QT += core gui widgets`）或第三方库。

### 2. 项目配置与生成
- **对于文本编码**：
  - 确保项目文件使用 UTF-8 编码，避免因编码问题导致的编译错误。
  - 在Windows下，要在头文件中增加 `#pragma execution_character_set("utf-8")` 来确保源文件以 UTF-8 编码编译。
- **对于 qmake 项目**：
  1. **创建构建目录**：在项目根目录下创建 `build/<Qt版本号>_<Release或Debug>` 目录（例如 `build/5.15.2_Debug`）。如果存在类似`build/Desktop_Qt_5_15_2_MSVC2019_64bit-Debug` 这样的目录，是因为这些目录是QtCreator自行创建的，请直接使用，不需要自行额外创建。
  2. **在 Windows PowerShell 中运行 qmake**：
     ```powershell
     cmd /c "cd /d <构建目录路径> && ""<vcvarsall.bat路径>"" x64 && <qmake.exe路径> <.pro文件相对路径>"
     ```
     **示例**：
     ```powershell
     cmd /c "cd /d e:\project\build\5.15.2_Debug && ""C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvarsall.bat"" x64 && D:\Qt5.15\5.15.2\msvc2019_64\bin\qmake.exe ..\..\project.pro"
     ```
  3. **检查输出**：确保没有出现 "Project ERROR" 之类的消息，成功后会生成 `Makefile`、`Makefile.Debug`、`Makefile.Release` 和 `.qmake.stash` 文件。
- **对于 CMake 项目**：
  1. 建议创建一个独立的构建目录（如 `build/`）。
  2. 指导用户运行 `cmake .. -DCMAKE_PREFIX_PATH=<path_to_qt>` 来配置项目，其中 `<path_to_qt>` 是 Qt 的安装路径（例如 `/opt/Qt/5.15.2/gcc_64/lib/cmake`）。
  3. 检查 CMake 输出，确认找到了正确的 Qt 版本和所需组件。

### 3. 编译与构建
- **使用 jom 编译**：在 Windows 环境下使用 jom 进行多核并行编译，速度更快。
  **PowerShell 命令**：
  ```powershell
  cmd /c "cd /d <构建目录路径> && ""<vcvarsall.bat路径>"" x64 && <jom.exe路径>"
  ```
  **示例**：
  ```powershell
  cmd /c "cd /d e:\project\build\5.15.2_Debug && ""C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvarsall.bat"" x64 && D:\Qt5.15\Tools\QtCreator\bin\jom\jom.exe"
  ```
- **检查编译输出**：jom 会自动选择合适的 Makefile（Release 或 Debug），编译成功后可执行文件位于 `release/` 或 `debug/` 子目录中。

### 4. 运行与测试
- **设置运行时环境**：在 PowerShell 中，需要将 Qt 的 `bin` 目录添加到 `PATH` 环境变量中：
  ```powershell
  $env:PATH = "D:\Qt5.15\5.15.2\msvc2019_64\bin;" + $env:PATH
  ```
- **处理第三方库依赖**：
  1. **从 .pro 文件中提取库路径**：检查 `.pro` 文件中的 `LIBS` 配置，识别第三方库的路径。
     - 例如：`LIBS += -L/root/Desktop/robot/rttr/build/install` 或 `LIBS += -LD:\libs\opencv\build\x64\vc16\lib`
  2. **确定动态库位置**：第三方库的动态库文件（.dll 或 .so）通常位于：
     - `-L` 参数指定的目录本身
     - 或该目录下的 `bin/` 子目录
     - 例如：`D:\libs\opencv\build\x64\vc16\lib` → 动态库可能在 `D:\libs\opencv\build\x64\vc16\bin`
  3. **添加到 PATH 环境变量**：
     ```powershell
     # Windows PowerShell 示例
     $env:PATH = "D:\Qt5.15\5.15.2\msvc2019_64\bin;D:\libs\opencv\build\x64\vc16\bin;D:\libs\assimp\bin;" + $env:PATH
     ```
     ```bash
     # Linux/macOS 示例
     export LD_LIBRARY_PATH=/usr/local/lib:/opt/opencv/lib:$LD_LIBRARY_PATH
     ```
  4. **自动识别策略**：
     - 解析 `.pro` 文件中所有 `LIBS += -L<path>` 条目
     - 对于每个路径，检查该路径及其 `../bin` 或 `./bin` 子目录是否存在动态库文件
     - 将找到的路径添加到 `PATH`（Windows）或 `LD_LIBRARY_PATH`（Linux）环境变量
- **运行可执行文件**：进入构建目录的 `release/` 或 `debug/` 子目录，运行生成的 `.exe` 文件。
  **示例**：
  ```powershell
  cd e:\project\build\5.15.2_Debug\release
  .\project.exe
  ```
- **完整运行命令（包含第三方库）**：
  ```powershell
  # Windows PowerShell
  $env:PATH = "D:\Qt5.15\5.15.2\msvc2019_64\bin;D:\libs\opencv\build\x64\vc16\bin;D:\libs\assimp\bin;" + $env:PATH; cd <构建目录>\release; .\<应用名>.exe
  ```
  ```bash
  # Linux
  export LD_LIBRARY_PATH=/opt/Qt/5.15.2/gcc_64/lib:/usr/local/opencv/lib:$LD_LIBRARY_PATH && cd <构建目录> && ./<应用名>
  ```

### 5. 调试支持
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

### Windows PowerShell 环境

| 任务 | 命令示例 |
| :--- | :--- |
| **查询 Qt 版本** | `& D:\Qt5.15\5.15.2\msvc2019_64\bin\qmake.exe --version` |
| **生成 Makefile (qmake)** | `cmd /c "cd /d <构建目录> && ""C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvarsall.bat"" x64 && D:\Qt5.15\5.15.2\msvc2019_64\bin\qmake.exe ..\..\project.pro"` |
| **编译项目 (jom)** | `cmd /c "cd /d <构建目录> && ""C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvarsall.bat"" x64 && D:\Qt5.15\Tools\QtCreator\bin\jom\jom.exe"` |
| **运行程序** | `$env:PATH = "D:\Qt5.15\5.15.2\msvc2019_64\bin;" + $env:PATH; .\release\project.exe` |

### Linux/macOS 环境

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
5.  **"UI 文件未编译"**：确保 `.pro` 文件包含 `FORMS += mydialog.ui`，或 CMake 中使用了 `qt_wrap_ui()`。
6.  **"Cannot run compiler 'cl'"**：这是因为 MSVC 编译器环境未初始化。必须在同一个命令会话中先运行 `vcvarsall.bat`，然后再运行 qmake 或编译命令。使用 `cmd /c` 确保所有命令在同一个会话中执行。
7.  **"找不到 Qt5Core.dll"**：运行时缺少 Qt 动态库。需要将 Qt 的 `bin` 目录添加到 `PATH` 环境变量中，或者将所需 DLL 复制到可执行文件目录。
8.  **"找不到第三方库的 DLL"**（如 opencv_world455.dll、assimp.dll 等）：
    - 从 `.pro` 文件的 `LIBS += -L<path>` 中提取库路径
    - 检查该路径或其 `bin/` 子目录是否包含所需的动态库文件
    - 将动态库路径添加到 `PATH` 环境变量（Windows）或 `LD_LIBRARY_PATH`（Linux）
    - 或将动态库文件复制到可执行文件所在目录
9.  **"程序运行时出现 0xc000007b 错误"**：通常是 32 位/64 位混用问题。确保 Qt、编译器、第三方库的架构（x86/x64）保持一致。

始终以清晰、分步骤的方式向用户解释流程和命令，并在执行任何可能修改文件的命令前获得用户确认。