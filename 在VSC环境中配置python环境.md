---
title: 在VSC环境中配置Python
date: 2021-01-24
top: true
tags: [python, 电脑小技巧]
categories: 计算机配置
mathjax: true
---

## <center>在VSC环境中配置python环境</center>

### 0. 前期配置

1. 安装python（**建议直接安装在系统目录下并勾选'ADD TO PATH'，避免不必要的麻烦**）
2. 安装Visual Studio Code（官网速度时快时慢，建议随缘下载，或者用可靠的梯子下载）
3. 检验python和VSC安装好了没
   * python: 在cmd或者terminal中输入python --version，如果能够输出版本号则证明安装成功（当前最新版本：3.9.1）
   * pip: python当前主流的包管理工具，在最新版本中pip会随python一同安装并完成系统变量设置，在cmd或者terminal中输入pip -V[--version]，若能正确输出版本号则证明安装成功

### 1. 建立虚拟环境(venv)

venv的作用可以理解为为一个特定的项目建立的一个python环境，可以理解为一个孩子。我们开发该项目时只需对该venv进行操作，venv出问题不会对全局造成影响。要建立venv，你需要：

1. 在合适的路径下建立一个文件夹（不妨设置为venv-demo）

2. 在该文件夹下运行terminal或者cmd，输入以下命令

   `python -m venv env`

   然后就会在该文件夹下生成一个名为env的文件夹，即为我们生成的虚拟环境。

3. 在terminal或cmd中输入以下命令

   `Set-ExecutionPolicy -Scope CurrentUser remotesigned`

   将当前用户设置为可以运行wps脚本。

   也可以通过以管理员身份运行wps，将-Scope属性去掉将全体用户设置为可运行wps脚本（之前设置过的可以忽略第3步）

4. 进入env文件夹执行以下命令

   `.\Scripts\activate`

   即可开启虚拟环境。输入 `exit` 即可退出。

### 2. 配置pip

由于众所周知的原因，大陆地区不修改pip的源pip的install速度会十分感人。所以我们需要对虚拟环境中的pip进行配置。

1. 在开启虚拟环境前提下，输入 `pip config debug` 查看是否有相应的pip配置文件，我们只需要建立虚拟环境下的pip配置文件即可

2. 在env文件夹下建立pip.ini文件

3. 打开pip.ini文件进行如下设置：

   ```ini
   [global]
   index-url = https://mirrors.aliyun.com/pypi/simple
   [install]
   trusted-host = mirrors.aliyun.com
   ```

   国内有许多商业公司和大学建立了pip的镜像站，个人认为阿里是这里面做得最好的，分别将index-url和trusted-host设置好，运行 `pip install [module_name]` 下载速度就可以接受了。

### 3. 设置VSC

1. 用VSC打开venv-demo文件夹
2. 这时VSC可能会提醒你安装pylint这个包，安装即可
3. 点击 `ctrl+shift+p` 打开settings面板，搜索 `Python:Select Interpreter` 选项，选择虚拟环境env的python.exe作为解释器
4. 这时点击 `ctrl+ ~ ` 打开终端，可以看到自动进入了env的虚拟环境
5. 在venv-demo建立demo.py文件，随便写点什么，然后在终端输入 `python demo.py` 可以正常运行，配置成功。 

