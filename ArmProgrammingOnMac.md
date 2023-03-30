# Arm Programming on Mac (Apple Silicon)

## Introduction

This document describes how to set up a development environment for Arm programming on Mac (Apple Silicon).

## Prerequisites
- [Xcode](https://developer.apple.com/xcode/) 12.2 or later
- [LLVM](https://llvm.org/) 11.0 or later
- [Clang](https://clang.llvm.org/) 11.0 or later
- [LLDB](https://lldb.llvm.org/) 11.0 or later

## How to make sure where the command line tools are installed

You can use this command:

```
xcode-select -p
```

By default, it should be `/Library/Developer/CommandLineTools`.

## How to compile the assembly language programs