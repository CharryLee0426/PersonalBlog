## Ubuntu 服务器上设置 MySQL

> 事情是这样的：之前开了一台运行 Ubuntu 的阿里云服务器，然后就安装了 MySQL，当时看到了 MySQL 成功登入的提示时就以为已经配置成功了。所以就没再去管它。现在我要将我的博客系统由之前 hexo 静态框架换成自己写的 SpringBoot 前后端分离的系统，在起项目时，就遇到了数据库配置错误的问题。于是开始了对 MySQL 在 Ubuntu 服务器上的安装的学习。

### 1. MySQL 在 Ubuntu 服务器上的安装

首先，对于一台新的服务器，如果是云服务器的话可以省略掉换源的步骤，必须先更新包管理器的信息。输入如下指令更新。

```shell
$ sudo apt-get update
```

云服务器默认是在 sudo 身份下运行，其实不加 sudo 一般也没有什么大问题。

然后开始安装 mysql-server 和 mysql-client。

```shell
$ sudo apt-get install mysql-server mysql-client
```

等待一段时间后安装完成，然后执行 `mysql --version` 检查是否安装成功。显示出版本信息即为成功。

### 2. MySQL 在 Ubuntu 上可能会出现的问题

当安装完 mysql 以后，会发现直接输入 mysql，一样可以进入 mysql。这样子是不行的。一般来说，我们部署SpringBoot项目如果没有用云数据库的话，大部分连接数据库以后是用 root 身份连接的，这个 root 身份必须有密码，否则在初始化项目的时候会报出这个错误：

```bash
ERROR java.SQLException: Access denied for 'root'@'localhost'
```

出现这个错误就说明数据库的用户配置有问题。但是网上关于这个错误的有价值的信息还是不多，大部分技术贴研究的错误是这个：

```bash
ERROR java.SQLException: Access denied for 'root'@'localhost' (using password: No[yes])
```

这个错误很大可能是项目的 `application.yml` 里的数据库信息（url, username, password）填错了，改了以后重新 package 一下部署就好了。

但上面的错误就不是改`application.yml`能行的了，这是因为云服务器的 MySQL 的配置有问题。

### 3. MySQL 用户设置

> 下面操作方法仅在 Ubuntu 20.04 LTS 测试，MySQL 版本号 8.0.26-0ubuntu0.20.04.2 for Linux on x86_64 ((Ubuntu))

首先，我们要找到 MySQL 的系统配置文件 `debian.conf`，文件路径在 `/etc/mysql`。然后用 cat 命令浏览该文件。该文件的基本格式如下：

![](https://pic.imgdb.cn/item/6194be372ab3f51d9156aa37.png)



这里提示了 **DO NOT TOUCH**，那就一下也别动，这个文件记录的就是该服务器上 MySQL 系统级用户的信息，如果这个文件丢了，那就只能重装了，GG。然后用这个账号登录 MySQL。

```shell
$ mysql -u debian-sys-maint -p
password: [yourpassword]
```

到这里就可以开始修改密码了。

> 这里开始的内容适用于 MySQL 8.0 即以上版本，对于老版本，网上有很多资料。

MySQL 8.0 后对账户系统作出了改变，原先的 Password 字段和 Password 函数都被弃用，密码所需的安全层级也提高了，设定的密码必须包含数字，大小写字母和特殊字符，长度须在8位以上。

在修改密码前，可以先选中 mysql 数据库，然后查看用户数据。

```mysql
SELECT user, plugin, authentication_string
FROM user;
```

可以看到 root 的 authentication_string 字段为空，plugin 为 ‘auto_socket’。这些都需要与上面的几个系统用户的格式保持一致，这时候可以执行如下语句修改密码和 plugin。

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'yourpassword';
```

这样就设置好了密码。

当然，这个密码是很复杂的，如果是个人项目，安全性要求没那么高的话，完全可以设置简单密码。在 MySQL 环境下，执行如下命令查看密码安全性策略。

```mysql
SHOW VARIABLES LIKE 'validate_password%';
```

然后修改其中的 `validate_password.policy` 和 `validate_password.length` 这两个属性的值，执行如下指令：

```mysql
SET GLOBAL validate_password.policy=0;	# 0=low, 1=midium, 2=strong
SET GLOBAL validate_password.length=1;	# minlength=4
```

这样就重新设置好了密码安全策略，MySQL 可以重新接受简单密码了。最后通过上面的命令设置简单密码。

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'yourpassword';
```

设置完毕后，退出系统账户，重新通过 root 进入 MySQL，此时已经无法通过无密码或者无用户方式进入。

### 4. 一点思考

其实这个错误一点都不难解决，但我还是花了好长时间，走了好多弯路，甚至拉着大佬走了弯路。归根到底还是我自己的技术积累不够，还得要多积累技术。**Talk is cheap, show me your code. **  多写代码，不能做思想上的巨人，行动的矮子。

再者，以后遇到问题了第一求证源不能是 CSDN 或者博客园了，这两个社区现在环境太差了，什么虫豸都能去写文章！多去 StackOverflow 和 Goole 上查解决方法，提高英语水平。

最后，一点关于 Linux 和一些开源框架的抱怨。既然我们可以做出很好用的产品，为什么开发者的工具就得这么难用？这个 Linux，总是会出各种各样的问题，现在一点也没体会到它的稳定和其他码农和知乎er说的“好用”。还有框架，就是 SpringBoot 和 其他软件比如 MySQL，原来的方法好好的，设置用户的命令好好的，你乱改它干什么！也不是什么大版本，就是两个小版本之间就可能有许多方法和接口的变动，导致项目之间的借鉴性很差。以前会很追求用新版本的开发工具，现在明白了，还是 Java8 这种稳定的老工具香。