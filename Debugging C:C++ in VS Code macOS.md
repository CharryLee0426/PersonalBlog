# Debugging C/C++ in VS Code macOS

## 0. Brief Introduction
Maybe VS Code is the most popular code editor(not IDE). When you first get a MacBook, the first thing you should do is
to install VS Code. If you want to run and debug C/C++ programs in VS Code, you will surely face some errors. It is
possible that you are not a blue hand and you can config `launch.json` and `tasks.json` quickly. However, everything changed since Apple silicon released. There is no doubt that Apple M1 is a revolutionary SoC, it has a remarkable performance. However,
there are many software having not been well compatible with AArch64, such as Docker and VS Code(although it has Apple silicon edition). This article will help you run and debug your C/C++ programs on your MacBook M1.

## 1. Prerequests
For the reason that VS Code is not an IDE, you should install C/C++ compiler in your computer first. By default, MacBook has own compiler called `clang`, which is very similar with `gcc/g++`. However, if you don't have it, you can install it quickly by this command. The command will help you download all the Xcode developer tools you will need.
```shell
$ xcode-select --install
```

You should also need the Microsoft official extension `C/C++` in the extension marketplace.

## 2. Create A Folder & Writting A Test `.cpp` File
That's so easy for you. You can do it very quickly.

```shell
$ cd ~/Desktop      # it's optional, default path is /Users/<yourName>
$ mkdir Test        # you can change the folder name to whatever you want
$ cd Test
$ touch test.cpp    # you can change the cpp name to whatever you want
$ code .
```

> VS Code has not installed `code` to your `$PATH`, so it's possible that 'code' command is not available for you. To make it available, you should open the VS Code and press `⌘ + ⇧ + P` and find `Shell Command: Install 'code' command in PATH` to install. The system will alert you that 'VS Code want to add something to your PATH', just allow it. Then go to your terminal and test code by using this command `which code`.

Your `.cpp` file can be very easy and even `hello world!`. You can use this `.cpp` as example.

```cpp
#include<stdio.h>

int main() {
    printf("Hello World.\n");
    return 0;
}
```

## 3. Config Your `launch.json` And `tasks.json`
First, you should find `Run -> Add Configution...`. Please make sure that `.cpp` file is your current active file or VS Code can't find current default configution set. Of course, you should choose `clang++ - Build and debug active file`. Then you will see a `.vscode` folder. In this folder, there are two configution files called `launch.json` and `tasks.json`. `tasks.json` tells VS Code that how to compile the `.cpp` file. `launch.json` is resposible for the debugging.

For those users who use MacBook on Intel chips, they can easily config all just by following the official guide. However, when you config all in the official guide on your MacBook M1, you just see the error that `assumed x86_64`. That's because VS Code devTools does not support AArch64 well.

There are two solutions:
1. compile and debug progaram in x86_64 (rosseta 2 will help you);
2. build universal(both available in x86_64 and Aarch64) binary.

I think that the first solution is more easy.

First, you should go to the `launch.json` and add x86_64 debugging config.

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "clang++ - Build and debug active file",
            // Above Does not Change.

            // Add This Line
            "targetArchitecture": "x86_64",
            
            // Below Does not Change.
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "osx": {
                "MIMode": "lldb",
                "preLaunchTask": "C/C++: clang++ build active file"
            }
        }
    ]
}
```
The `targetArchitecture` tells VS Code that which architecture the target binary should in.

Then go to the `tasks.json`, do the following change.

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: clang++ build active file",
            "command": "/usr/bin/clang++",
            "args": [
                "-fdiagnostics-color=always",
                // Above Do not Change 
                
                // Add this Line
                "-arch", "x86_64",

                // Below Do not Change
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
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
            "detail": "Task generated by Debugger."
        }
    ],
    "version": "2.0.0"
}
```

You add `-arch` into args so that when you start debugging it can compile to x86_64 binary.

Then you can debug C/C++ successfully.

## 4. Debugging `scanf` Function
It's easy, just set `externalConsole` in `launch.json` to true. You can test it by using this example `.cpp` file.

```cpp
#include<stdio.h>

int main() {
    int a = 0;
    printf("The default value of a is %d\n", a);    // 0
    scanf("%d", &a);
    printf("Now a = %d\n", a);
    return 0;
}
```