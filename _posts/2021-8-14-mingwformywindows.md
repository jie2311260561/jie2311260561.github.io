---
layout: post
title:  "mingw 在我电脑上的部署"
date:   2021-8-14
categories: tools
tags: gcc mingw 
excerpt: mingw安装在自己电脑使用gcc工具
--- 

# 前言
昨天CVTE笔试感觉自己还行了？今天大疆在公司的笔试才发现自己的代码，已经基本生疏了，已经不怎么会写基础的C语言的程序了，简简单单的两个题目就写了很久，实在是不应该！！！

# 所以
所以我要在我自己电脑上安装gcc工具，没事就练练C语言，真的是应了那句话，代码几天不打就会生疏！！！




# 除了 Dev 的
我在自己电脑上还发现了之前安装的TouchGFx 也有Mingw 的编译工具链，比Dev 的版本更新一点，毕竟Dev 用的已经是2014年的Gcc 了 
遗憾的是 这个 Mingw 的 target 不是 w64 的有点遗憾啊
 

从网上再找的安装一个 或者看看 Qt  看看
Qt 的gcc 版本gcc version 5.3.0 (i686-posix-dwarf-rev0, Built by MinGW-W64 project)  感觉还是不太行  找找新玩玩

下载好了  这比较新 估计比较好用，不说了 先用着玩 ~
gcc version 8.1.0 (x86_64-posix-seh-rev0, Built by MinGW-W64 project)

下载链接！https://sourceforge.net/projects/mingw-w64/files/

环境变量添加好！  重启 vscode 嘻嘻！ 


# 准备vscode 的环境


task.json
```
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "command": "g++",
            "args": ["-g","${file}","-o","${fileDirname}\\${fileBasenameNoExtension}.exe"],    // 编译命令参数
            "problemMatcher": {
                "owner": "cpp",
                "fileLocation": ["relative", "${workspaceRoot}"],
                "pattern": {
                    "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            },
            
        }
    ]
}
```



launch.json

```
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "C/C++ Runner: Debug Session",
      "type": "cppdbg",
      "request": "launch",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${fileDirname}",
      "environment": [],
      "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
      "internalConsoleOptions": "openOnSessionStart",
      "preLaunchTask": "build",
      "MIMode": "gdb",
      "miDebuggerPath": "D:\\Program Files (x86)\\Dev-Cpp\\MinGW64\\bin\\gdb.exe",
      "externalConsole": true,
      "setupCommands": [
        {
            "description": "Enable pretty-printing for gdb",
            "text": "-enable-pretty-printing",
            "ignoreFailures": true
        }
      ]
    }
  ]
}
```
