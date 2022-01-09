---
title: Git Basic Tutorial
date: 2021-04-15
tag: 团队协作技术
categories: 计算机配置
top: true
mathjax: false
---

# Git Basic Tutorial

> 这个教程用于介绍如何使用 git 进行版本控制和团队协作。
>
> Author: LC
>
> Date: 2021-04-15



---



### 1. 下载 git 并配置好环境变量

这一步应该没什么好说的



### 2. 注册 gitee 账号

gitee 是中国的 git 代码托管平台，网速比较稳定。注意：**要把邮箱绑定了，之后有用**



### 3. 配置 git 信息

在命令行下输入如下命令

```
git config --global user.name "yourname"
git config --global user.email "youremail"
```

这里的 emai 就是第 2 步中绑定的邮箱。



### 4. Fork 本仓库

在 web 端到本仓库首页，可以看到 fork 按钮，此时就可以将本仓库 fork 到自己的账户上。fork 这个概念相当于把本仓库 copy 到自己的账号下。

对于第一次建立的仓库，应该首先用 git 工具初始化在目录下生成 .ignore 文件。

```bash
git init
```



### 5. clone 第 4 步 fork后的仓库到本地



### 6. 配置上游URL

输入命令

```bash
git remote add upstream [URL]
```

因为自己提交代码最后要提交到 「上游」，也就是被 fork 的那个仓库，所以需要进行如下配置。

这个 URL 可以在原仓库的地址栏处找到。

输入命令

```bash
git remote -v
```

可以查看 origin 和 upstream 是否配置成功。



### 7. 开始工作

**开始工作前首先要保证本地和 master 分支保持和最新同步**

输入命令

```bash
git pull --rebase upstream master
```

即可保持同步



**注意，不要在 master 分支进行修改**

因为我们本地的 master 分支要和上游 master 分支保持同步和通信，直接修改master 分支容易发生冲突，所以我们要创建一个新的分支。

输入命令

```bash
git checkout -b yourbranchname
```

创建新分支，分支名尽量有意义，虽然可以随便起。



### 8. 开始本地文件的修改

正式的工作内容，比如代码，写文档。



### 9. 提交 PR

输入命令

```bash
git status
```

可以查询更改状态。比如新建文件或文件夹以后刚开始会显示 untracked （不能追踪到）。删除文件或文本文档（也就是代码的变动都可以看出来）。

输入命令

```bash
git add .
```

将变动添加到暂存区（可以理解为缓存）。提交分为两个部分：1. 本地到暂存区；2. 暂存区到远程仓库分支。add 命令后的「 . 」可以理解为通配符，即将所有的变化统一提交到暂存区。

输入命令

```bash
git commit -s -m "yourcommentforthiscommitment"
```

`-s` 可以将 name 和 email 附在提交中，方便追踪管理，最后的字符串可以简要描述此次提交的目的，解决的问题。

输入命令

```bash
git push -u origin yourbranch 
```

将你的 commit 提交到 fork 仓库的非 master 分支（也就是之前你创建的分支）

提交工作完成！



### 10. Compare & pull request

如果前 9 步都没有问题的话，此时你的仓库会受到你提交的信息。此时进入 web 端进行操作。点击分支会收到刚刚提交的分支。然后进行比对，检查无误后就可以点击 **pull request** 进行提交了。我们一般提交到上游的 master 分支。



### 11. 删除分支

```bash
git checkout branchname
```

切换分支

```bash
git branch -a
```

列出所有分支

```bash
git checkout -d branchname
```

删除分支

```bash
git push origin --delete branchname
```

删除远程仓库的分支

### 12. 新增：关于 git 的流程

![](https://pic.imgdb.cn/item/61b9d4cf2ab3f51d9168a582.png)

```mermaid
graph LR

b--Pull Request-->a
a--fork-->b
c["local/repo"]
d["local/repo"]
b--git clone-->c
c--"origin"-->b
d--upstream-->a
a--git pull-->d
subgraph github
a["dalao/repo"]
b["you/repo"]
end
```

如图，一般情况下，默认自己为参与者，可以将 git 的流程看成三个仓库，`dalao/repo`, `you/repo`, `local/repo`。fork 不是原生 git 的概念，可以将 fork 当成是 GitHub 服务器上的 git clone。所以在本地项目中要参与其他人的项目（没有被认证为 contributor），不能直接向上游推 commit。而是要先推上自己的项目。然后向上游提交 Pull Request（这通常由）。由上游仓库的管理员决定是否接受你的 Pull Request。所以一般的项目要设置两个远程仓库，通过以下命令添加：

```bash
git remote add origin you/repo.git	# 设置 github 端的源为自己 fork 仓库的 URL
git remote add upstream dalao/repo.git # 设置 github 端的上游为创始者仓库的 URL
```

### 13. 新增：关于 git pull 和 git reset 的区别

 为什么 git pull 不能将自己在本地删除的文件恢复

因为 git pull 会保存自己在本地的修改，而另外一个命令 `git reset` 可以将仓库完全重置到远程仓库的某一次 commit 上，并且不会保留自己的修改。

例如，在本地，仓库中有两个 commits，A 和 B，在远程仓库中有两个 commit，A 和 C。这样的话执行如下命令

```bash
git pull
# A, B, C
git reset
# A, C
```

### 14. 新增：如何配置本地的 macOS GitHub环境

感觉虽然 gitee 也不错，但还是 GitHub 的体验更好一些。但是一些众所周知的原因 GitHub 的访问体验在大陆很不好，所以使用前要提前设置一下。

需要提前说明的是，要自己准备好科学上网手段，这里大家自由发挥，我是从一个看片老哥手里拿到的稳定ss。我们需要设置 git 工具走 shadowsocks 代理，要给 git 一个全局变量。

```bash
git config --global http.proxy "socks5://127.0.0.1:1086" # macOS 默认是 1086 端口
git config --global https.proxy "socks5://127.0.0.1:1086"
```

然后就可以正常使用了，不过这样有一个缺点，就是 git 工具必须得在 shadowsocks 开启的情况下才可以使用。一但 shadowsocks 节点/订阅挂掉就失去了与世界先进思想交流的机会。
