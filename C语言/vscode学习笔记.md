[toc]
# Predefined variables
参考:(https://code.visualstudio.com/docs/cpp/config-mingw)

|变量名 |含义 |
| --- | --- |
${workspaceFolder}| - the path of the folder opened in VS Code
${workspaceFolderBasename}| - the name of the folder opened in VS Code without any slashes (/)
${file} |- the current opened file
${fileWorkspaceFolder}| - the current opened file's workspace folder
${relativeFile}| - the current opened file relative to workspaceFolder
${relativeFileDirname}| - the current opened file's dirname relative to workspaceFolder
${fileBasename}| - the current opened file's basename
${fileBasenameNoExtension}| - the current opened file's basename with no file extension
${fileDirname}| - the current opened file's dirname
${fileExtname}| - the current opened file's extension
${cwd}| - the task runner's current working directory upon the startup of VS Code
${lineNumber}| - the current selected line number in the active file
${selectedText}| - the current selected text in the active file
${execPath}| - the path to the running VS Code executable
${defaultBuildTask}| - the name of the default build task
${pathSeparator}| - the character used by the operating system to separate components in file paths

# 创建调试环境
[链接](https://zhuanlan.zhihu.com/p/419473501)
打开vscode，在里面创建一个工程，工程下面分别创建以下几个文件夹“：app;bin;src;inc;doc”

- app文件夹：只用来存放main.c文件，

- bin文件夹：用来存放编译好的可执行的.exe二进制文件

- src文件夹：用来存放bsp文件，也就是各个功能模块的源文件

- inc文件夹：用来存放各个功能模块的头文件

- doc文件夹：用来存放项目工程的说明文档

## 测试程序的编写
首先我们在app文件夹下面创建main.c文件，文件里面的代码如下所示：
```c
#include "stdio.h"
#include "stdlib.h"
#include "string.h"
#include "../inc/app_test.h"

int main()
{
    char str[] = "app_test_print function!";

    printf("this is first c project!\r\n");

    app_test_print(str);

    system("pause");

    return 0;
}
```
src文件夹中创建app_test.c文件，app _test.c文件中的内容如下所示：
```c
#include "stdio.h"
#include "stdlib.h"
#include "string.h"
#include "../inc/app_test.h"

void app_test_print(char *_Src)
{
    printf("%s\r\n", _Src);
}
```
在inc文件夹中创建app_test.h文件。app_test.h文件中的内容如下所示：
```c
#ifndef APP_TEST_H
#define APP_TEST_H

void app_test_print(char *_Src);

#endif
```
## launch.json
参考：https://www.cnblogs.com/zsh-notes/p/14514745.html
这个配置文件是告诉vscode如何来启动调试你的代码程序的，这其中包括你的程序在哪个位置，你用什么工具来调试，调试的时候需要给调试工具传什么参数等。
- program：代表要运行的二进制文件（也就是你的C代码编译出来的程序）所在路径。
- miDebuggerPath：代表调试器（GDB）所在路径。
- preLaunchTask：在运行program前要做的前置任务，比如编译，task.json就是用于定义前置任务。

实例：
```json
{
    // 使用 IntelliSense 了解相关属性。
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "gcc.exe - 生成和调试活动文件",
            "type": "cppdbg",
            "request": "launch",
            "program": "${cwd}\\bin\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "D:\\SoftWare\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: gcc.exe 生成活动文件"
        }
    ]
}

```

![image](https://img2020.cnblogs.com/blog/1405118/202107/1405118-20210715231158321-2001731676.png)


## tasks.json
 这个配置文件是用来执行你预定的任务的，比如说你修改了你的代码，调试之前，肯定要重新生成新的程序后再调试，那么你就可以配置它告诉vscode怎么重新生成这个新的程序。(task.json不是必须文件，比如python调试，可以不用提前编译)

- label：指定前置任务（比如：“C/C++: gcc 生成活动文件”）名称
- command：任务执行命令，一般来说执行编译命令：gcc
- args：用于command后面的参数，比如：-g（debug选项），-f等

实例：
```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: gcc.exe 生成活动文件",
            "command": "D:\\SoftWare\\mingw64\\bin\\gcc.exe",
            "args": [
                "-g",
                // "${file}",
                "${cwd}\\src\\*.c",
                "${cwd}\\app\\*.c",
                "-o",
                "${cwd}\\bin\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```
![image](https://img2020.cnblogs.com/blog/1405118/202107/1405118-20210715231423303-49912544.png)

vscode就是先跑 tasks.json 任务，再跑 launch.json。
按照上述配置后，可以调试，但如果点运行还是不行，原因是run coder插件的问题，上述两个文件似乎对run coder不起作用。

# run coder 设置编译多个源文件
参考：https://blog.csdn.net/weixin_45912291/article/details/121771097
在设置中搜索 Code-runner: Executor Map
打开settings.json编辑，将行内的 g++ $fileName 改为 g++ *.cpp 即可
![image](https://img-blog.csdnimg.cn/img_convert/b700423035a33775e71be8a0f9ad246f.png)

# 其他配置文件
## settings.json
把这个文件里的东西放到“用户设置”里可以覆盖全局设置，否则只在当前工作区才有效。这两点各有自己的优势。
```json
作者：谭九鼎
链接：https://www.zhihu.com/question/30315894/answer/154979413
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

{
    "files.defaultLanguage": "c", // ctrl+N新建文件后默认的语言
    "editor.formatOnType": true,  // 输入分号(C/C++的语句结束标识)后自动格式化当前这一行的代码
    "editor.suggest.snippetsPreventQuickSuggestions": false, // clangd的snippets有很多的跳转点，不用这个就必须手动触发Intellisense了
    "editor.acceptSuggestionOnEnter": "off", // 我个人的习惯，按回车时一定是真正的换行，只有tab才会接受Intellisense
    // "editor.snippetSuggestions": "top", // （可选）snippets显示在补全列表顶端，默认是inline

    "code-runner.runInTerminal": true, // 设置成false会在“输出”中输出，无法输入
    "code-runner.executorMap": {
        "c": "gcc '$fileName' -o '$fileNameWithoutExt.exe' -Wall -O2 -m64 -lm -static-libgcc -fexec-charset=GBK -D__USE_MINGW_ANSI_STDIO && &'./$fileNameWithoutExt.exe'",
        "cpp": "g++ '$fileName' -o '$fileNameWithoutExt.exe' -Wall -O2 -m64 -static-libgcc -fexec-charset=GBK && &'./$fileNameWithoutExt.exe'"
        // "c": "gcc $fileName -o $fileNameWithoutExt.exe -Wall -O2 -m64 -lm -static-libgcc -fexec-charset=GBK -D__USE_MINGW_ANSI_STDIO && $dir$fileNameWithoutExt.exe",
        // "cpp": "g++ $fileName -o $fileNameWithoutExt.exe -Wall -O2 -m64 -static-libgcc -fexec-charset=GBK && $dir$fileNameWithoutExt.exe"
    }, // 右键run code时运行的命令；未注释的仅适用于PowerShell（Win10默认）和pwsh，文件名中有空格也可以编译运行；注释掉的适用于cmd（win7默认）、PS和bash，但文件名中有空格时无法运行
    "code-runner.saveFileBeforeRun": true, // run code前保存
    "code-runner.preserveFocus": true,     // 若为false，run code后光标会聚焦到终端上。如果需要频繁输入数据可设为false
    "code-runner.clearPreviousOutput": false, // 每次run code前清空属于code runner的终端消息，默认false
    "code-runner.ignoreSelection": true,   // 默认为false，效果是鼠标选中一块代码后可以单独执行，但C是编译型语言，不适合这样用
    "code-runner.fileDirectoryAsCwd": true, // 将code runner终端的工作目录切换到文件目录再运行，对依赖cwd的程序产生影响；如果为false，executorMap要加cd $dir

    "C_Cpp.clang_format_sortIncludes": true, // 格式化时调整include的顺序（按字母排序）
}
```
## c_cpp_properties.json
如果你确定不需要使用别人的库，则现在的版本（0.18.0之后）不需要创建这个文件了，cpptools会自动使用默认的设置。所以本文也不再包含此文件的配置。如果你自己编写了头文件又不在workspaceFolder下，或是使用别人的库，就需要手动创建这个文件放到.vscode下了。模板可以参考：
https://link.zhihu.com/?target=https%3A//github.com/Microsoft/vscode-cpptools/blob/master/Documentation/LanguageServer/MinGW.md

## 调试
### gdbserver
launch.json
```json
{
    // 使用 IntelliSense 了解相关属性。
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
    {
    "name": "g++ - 生成和调试活动文件",
    "type": "cppdbg",
    "request": "launch",
    "program": "${workspaceFolder}/src/.libs/suricata",
    "args": [],
    "stopAtEntry": false,
    "cwd": "${workspaceFolder}/src/.libs/",
    "environment": [],
    "externalConsole": false,
    "MIMode": "gdb",
    "setupCommands": [
    {
    "description": "为 gdb 启用整齐打印",
    "text": "-enable-pretty-printing",
    "ignoreFailures": true
    },
    {
    "description": "将反汇编风格设置为 Intel",
    "text": "-gdb-set disassembly-flavor intel",
    "ignoreFailures": true
    }
    ],
    "preLaunchTask": "C/C++: g++ 生成活动文件",
    "miDebuggerPath": "/usr/bin/gdb",
    "miDebuggerServerAddress":"10.0.2.145:2000"
    }
    ]
    }
```
task.json
```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: gcc 生成活动文件",
            "command": "gdbserver @localhost:2000 ./src/.libs/suricata -c ./suricata.yaml -r /tmp/share/Work/packet/pop3/pop3-manyCC.pcap  -l /tmp/share/Work/suricata_output/ -S /tmp/share/Work/suricata_rules/rules.rules",
            "args": [
                
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}

```

不用gdbserver
launch.json
```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    // 调试使用文件
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++ - Build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/src/.libs/suricata",//需要运行的程序
            "args": [
                "-c",
                "${workspaceFolder}/suricata.yaml",
                "-l",
                "/tmp/share/work/suricata_output/",
                "-s",
                "/tmp/share/work/suricata_rules/manyrules.rules",
                "--runmode",
                "single",
                "-r",
                "/mnt/share/work/packet/smtp/smtp_wxjpg.pcap",
            ],//程序执行需要传入的参数
            "stopAtEntry": false,//true时，打开控制台以后停止，暂时不执行程序
            "cwd": "${workspaceFolder}",//当前工作路径
            "environment": [],
            "externalConsole": false,//使用外部控制台
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: make build project",//执行编译任务，内容对应task.json中label选项
            "miDebuggerPath": "/usr/bin/gdb"//调试指令
        }
    ]
}
```

tasks.json
```json
{
    //编译使用文件
    "tasks": [
        {
            "type": "process",
            "label": "C/C++: make build project",//任务名字，任务名字叫"c/c++:make project"，
                                          //与launch.json中的preLaunchTask对应
            "command": "make",//编译命令
            "args": [],//编译参数
            "problemMatcher": [
                "$gcc"//使用gcc捕获错误
            ],
            "options": {
                "cwd": "${workspaceFolder}"//vs code 打开的文件夹路径
            },
 
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "Task generated by Debugger."
        }
    ],
    "version": "2.0.0"
}
```
settings.json
```json
{
    "files.associations": {
        "array": "c",
        "functional": "c",
        "tuple": "c",
        "type_traits": "c",
        "utility": "c",
        "debug.h": "c",
        "suricata-common.h": "c",
        "thread": "c"
    }
}
```

### sftp
```json
{
    "name": "sftp_test",
    "host": "10.0.2.145",
    "protocol": "sftp",
    "port": 22,
    "username": "root",
    "password":"tfa@123",
    "remotePath": "/tmp/share/sftp_test",
    "uploadOnSave": true,
    "useTempFile": false,
    "openSsh": false,
    "ignore": [
        "**/.vscode/**",
        "**/.git/**",
        "**/*.o"
    ],
    "watcher": {
        "files": "*",
        "autoUpload": false,
        "autoDelete": false
    }
}

```