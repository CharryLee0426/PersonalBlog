# 在 Mac 上使用 Docker 和 k8s

## 0. 介绍

Docker 和 k8s 是云时代下两种容器类应用，相比之下，Docker 相对更简单一些，k8s 在企业中使用的更多一些，这几天的实习工作中，我具体学习了这两种容器工具的使用。

## 1. Docker

按照 mentor 的 roadmap，我是应该先看 k8s 的，但是 k8s 比 Docker 复杂的多，即使是用官网上的 minikube 进行入门教程的复现都会出现错误。所以我就先学习了 Docker。

Docker 简单来说就是虚拟机的进一步抽象化，相比起虚拟机，Docker 更加轻量，可以理解为一台虚拟机在结构上就是一台真正的电脑（只是硬件的部分被虚拟化了）而 Docker 只是一个为了运行某个程序的一个承载环境，更容易移植。

Docker 的主要命令有

```bash
docker build -t <tagname> .
# <tagname> is your iamge name, . means that you will search Dockerfile in the current directory for building.
```

这个命令是从源码按照 `Dockerfile` 的指示构建 Docker Image。

```bash
docker run -dp <host_port>:<container_port> <image_name>
# -d stands for depatched(running background), -p means that create a port mapping.
```

这个命令是从已经在 Docker daemon 中已经存在的 Image 创建一个 container。

Image 和 Container 的关系类似于类与实例的关系。一个 image 可以创建多个 container。不过多个 container 的文件系统不共享，即容器 A 作出的修改在容器 B 中并不生效。

```bash
docker ps
```

这个命令可以将现在正在运行的 container 列出。

```bash
docker exec <container_id> <command>
# command will run in your container.
```

这个命令适合在一些容器中执行一些命令。

为了让容器之间可以通信，我们可以用持久层技术实现。像 sqlite 这样的轻量级数据库，就建立一个 volume 的 .db 文件就好了。像 MySQL 这样的大型数据库，就需要开一个 network。

详细的可以看 Docker 的官网，有一个入门教程，很详细了。

## 2. k8s

K8s 光是配置环境就很麻烦了，minikube 还会有各种各样的错误。所以我使用了我们公司的集群做实验。使用 k8s port-forward 可以做端口转发，使得在本机上也能访问到公司的集群。

k8s 的重要概念有 pods，deployment，service等等。还要理解命令式 API 和指导式 API 的不同之处，以及 k8s 内部有一个机制让所有集群按照要求尽可能的处于想要的状态。

具体的可以参考文档。