# Debug C and CPP code in vs code

这里简要记录的在 vscode 中调试 c/c++ 代码的设置过程。
以 Linux 系统为例。

## 1. 依赖

### VS Code 插件依赖

+ codeLLDB [vadimcn.vscode-lldb](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb)
+ clangd [llvm-vs-code-extensions.vscode-clangd](https://marketplace.visualstudio.com/items?itemName=llvm-vs-code-extensions.vscode-clangd)
+ cmake [twxs.cmake](https://marketplace.visualstudio.com/items?itemName=twxs.cmake)
+ CMake Tools [CMake Tools]( https://marketplace.visualstudio.com/items?itemName=ms-vscode.cmake-tools )

注意：CodeLLDB 可能需要代理访问国际互联网。

### 工具依赖

#### lldb

codeLLDB 插件依赖于 lldb，所以你需要安装它在系统上。以 Ubuntu 为例：
`sudo apt install -y lldb'

#### Clangd

    什么是 Clangd: clangd is a language server that can work with many editors via a plugin. Here’s Visual Studio Code with the clangd plugin, demonstrating code completion.

![What's Clangd](https://clangd.llvm.org/screenshots/basic_completion.png)

对于 Clangd，其依赖于一个编译数据库，为 compile_commands.json。

对于 Clangd 的使用，需要在系统上安装 binary，以 Ubuntu 为例：
`sudo apt install -y clangd`

在 VS code 和 OS 都安装完 clangd 之后，你可以在 VS Code 的设置中对 Clangd 进行设置。打开设置，搜索 `clangd.arguments`,可以在添加 Clangd 的参数，比如：

+ `--clang_tidy` 启用 clang_tidy 的功能。更多功能你可以查看 Clangd 的文档
+ `--pretty` 启用美观的 json 输出
+ `--log=error` log 等级
+ `--query-driver=<string>`,你可以设置搜索头文件的编译器，比如 `--query-driver=/usr/bin/gcc*`
+ `--background-index` 后台索引代码，并持久化索引信息
+ `--fallback-style=<string>` 当工程根目录下不存在 `.clang_format` 时，format 的格式，比如，如果你偏好 Google Stype, 可以设置 `--fallback-style=Google`
  
你可以根据自己的需要，设置这些参数。更多的参数，可以 `man clangd` 查看。

如果你的工程由 Cmake 组织：

#### CMAKE

如果你的工程用 CMake 组织，那么可以一下两种方式传递给 CMAKE 定义

+ 在 Command Line 中：`cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1 ....`  
+ 在 CMakeLists.txt 中，添加 `set(CMAKE_EXPORT_COMPILE_COMMANDS ON)`

以生成 Clangd 需要的 compile_commands.json 文件。

如果你的工程有 make 组织：

#### bear

Bear 是一个为 clang 工具生成编译数据库的工具。
以 Ubuntu 为例：
`sudo apt install -y bear`

在 build 脚本中的 make 命令前，添加 `bear --`， 可以在当前目录生成 `compile_commands.json`：

```shell

# other cmd
bear -- make all
# go oning
```

这个方式对于仅有一个 Makefile 的工程是可以的，但对于具有多个 Makefile 的工程，活着具有多个文件夹，其会在不同的位置产生  `compile_commands.json`。由于 Clangd 不能同时处理多个编译数据库，所以体验上不是很好。
所以对于这种情况，最好的解决办法是使用 bear 的 append 功能和 make 的 `-C {$make directory}` 选项。举例而言：

```
./
 |-- a.c
 |-- Makefile
 |-- build.sh
 |--/libs
    |-- libtest.c
    |-- Makefile
    |-- build_libtest.sh
```

此时，通常的编译顺序是先编译 `libtest`，然后再编译 `a.c`，所以，我们再第一项编译的地方加入：

```shell

## build_libtest.sh
pushd 
bear --output /your/target/path/of/compile_commands.json -- make all
```

在编译 `a.c` 的时候，编译脚本里的 bear 需要 append 到这个上边指定的文件路径

```shell
## build.sh
bear --output /your/target/path/of/compile_commands.json --append -- make all
```
其他情况类型类似。最终原理是就后续的 bear make 都需要 append 到第一次生成的 compile_commands.json 中。

最终你会在 `/your/target/path/of/compile_commands.json` 看到 json 形式的编译数据库。

## 2. Debug

Linux 系统中 VS code 中 C/C++ 的 debug 依赖于 GDB/LLDB。假设已经在某个目录打开了 VS Code，那么点击 主侧栏 中的 debug 按钮，可以创建 launch.json 文件。最终会在当前目录的 .vscode/ 文件夹下创建 launch.json。类似如下：

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "main",
            "program": "${workspaceFolder}/a.out",
            "stopOnEntry": true,
            "args": [
                "-c",
                "myconfig.json"
            ],
            "cwd": "${workspaceFolder}",
            "preLaunchTask": "make example"
        }
    ]
}
```

**在参数键上悬停可以看注释。**

这里面有几个点需要注意：

+ 使用 codelldb 必须要设置 `type` 为 `lldb`
+ `request` 为 `launch` 或 `attach`
+ `program` 为调试对象的路径
+ 需要注意的是 `args` 数组元素是参数，空格分开就是一个单独的元素
+ `preLaunchTask` 可以设置在 debug开始前的一些工作，**注意这个值要与 tasks.json 中的 `lable` 对应。**

说一下 `preLaunchTask`, 可以在 debug 开始前，执行一些操作，比如：修改了一些源文件，重新make。你可以在工具栏 “终端”的配置任务里，配置任务，也可以直接在 .vscode/ 下创建 tasks.json （如果存在该文件，可以打开进行修改）：

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "make example",
            "type": "shell",
            "command": "cd ${workspaceFolder}/ffmpeg/doc/; make",
            "args": [
            ],
            "group": "build",
            "presentation": {
                "reveal": "always",
                "panel": "shared"
            },
        }
    ]
}
```

可以看出，在 `command` 中我进行了 make 操作。

更多的使用方式可以参考文档。

