# Docker

**这篇文章讨论了Docker的原理和基础用法**
<!--more-->
## 基础构架
* Docker使用系统本身使用的技术进行虚拟化.
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/docker/architecture.png "Architecture")
  * Docker Daemon
    * 运行在系统的一个应用.
    * 持续等待Docker API 相关的请求.
    * 建立管理 Container.
    * Docker API 以HTTP形式建立并且返回(Restful API).
  * Docker Client
    * Docker CLI.
    * 运行命令 etc. docker run, docker build...
## 与传统虚拟化技术的区别
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/docker/virtual.png "Difference")
* VM：
  * **VM** 需要为每一个虚拟机创建一个用户OS，许多二进制文件和库,大概需要10GBs的.
  * **VM** 启动很慢.
* Container:
  * Docker通过Linux Kernel的cgroup和namespace来对container进行环境隔离.
  * 每个Container都共享同一个kernel，所以如果一个container导致kernel崩溃后，其他所有container也会shutdown. 

## 容器
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/docker/container.png "Container")
* Docker会把每一个每一个容器赋予**namespace**并把容器与kernel的一个**cgroup**绑定.
* **cgroup** 帮助管理container里的资源(I/O speed, Memory Usage, number of process).

## Docker Image
* 用户可以通过 `docker pull image name:version` 来获得 **DockerFile**文件.
* DockerFile描述了Docker Image里面的环境和应用.
* Docker Image layer:
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/docker/layer.png "Docker Image Layer")
  * layer代表了DockerFile里面的一些指令.
  * DockerFile里的一些操作比如:`RUN`,`COPY`,`ADD` 都会创建一个layer.
  * 第一层是R/W层，也叫容器层.
  * 容器过程中对文件的修改，删除只会造成容器层的改变.
  * 后面的layer是不可修改的，是镜像层.
  * Docker Image layer的用处在同一个image启动多个container时，layer层是共享的，所以启动非常快.

## Docker Volume
* TODO


## 常用命令
* docker run
* docker start
* docker stop
* docker pull
* docker build
* docker ps
