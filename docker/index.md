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
* 用户可以通过 `docker build <path>` 来创建 docker image.
* DockerFile:
  * DockerFile描述了Docker Image里面的环境和应用.
  * `ADD` - add files from local to containers.
  * `FROM` - select a base image to build the new image on the top of.
  * `CMD` - command run when container starts.
  * `RUN` - Specify commands to make changes to your Image and subsequently the Containers started from this Image. 
            This includes updating packages, installing software, adding users, creating an initial database, setting up certificates, etc. 
* Docker Image layer:
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/docker/layer.png "Docker Image Layer")
  * layer代表了DockerFile里面的一些指令.
  * DockerFile里的一些操作比如:`RUN`,`COPY`,`ADD` 都会创建一个layer.
  * 第一层是R/W层，也叫容器层.
  * 容器过程中对文件的修改，删除只会造成容器层的改变.
  * 后面的layer是不可修改的，是镜像层.
  * Docker Image layer的用处在同一个image启动多个container时，layer层是共享的，所以启动非常快.

## Docker Volume
![Alt text](https://github.com/ArberSephirotheca/czy.github.io/raw/master/docker/volume.png "Docker Volume")
* Bind mounts:
  使用方法:
    * `docker run -v <HOST_PATH>:<CONTAINER_PATH> --name xxxx xxxx:version`
    * volume会把 `<HOST_PATH>`的内容映射到container里面的`<Container_PATH>` 当中.
  * 作用:
    * 对于需要不停修改的image， docker volume 让你不用重新修改DockerFile/build image, 只需要修改 docker volume/container里面的文件.
    * 使container的数据保存在 `var/lib/docker/volume` 下，即使container被删除，依旧有效，保持**数据持久化**.

* volume:
  * 使用方法:
    * `docker volume create my-vol`- Create a volume.
    * `docker volume inspect my-vol`- Inspect a volume.
    * `docker volume rm my-vol` - Remove a volume. 
    * `docker run -v <DOCKER_VOL>:<CONTAINER_PATH> --name xxxx xxxx:version`
  * 作用:
    * 使container之间可以**共享**volume的内容.
    * 保持**数据持久化**，只有volume被删除后内容才会消失.
  
* tmpfs(temporary file system):
  * 只适用于**linux**.    
  * 会分配一块**内存空间**，存放特定目录底下的容器档案资料.
  * 使用`stop`指令停用容器的时候，内存空间会被解放.
  * 使用方法:
    * `docker run --name <name> --tmpfs <CONTAINER_PATH> xxxx:version`
  * 作用:
    * 放在内存里，速度比放在disk里面更快.
## Docker Networking
* TODO
## 常用命令
* `docker run` - 下载并执行容器.
* `docker start` - 开始已有容器.
* `docker stop` - 关闭已有容器.
* `docker pull` - 下载容器.
* `docker build <DockerFile>` - 建立容器.
* `docker exec <name> <path>` - 执行容器里的特定程序如bash.
* `docker ps`

## 常用参数
* `-i` - run container in a interactive mode.
* `-t` - provide a tty for named container.
* `-d` - run in the background and return container ID.
