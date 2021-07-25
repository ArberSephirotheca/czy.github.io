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
    * `docker run -v <HOST_PATH>:<CONTAINER_PATH> --name <name> <image name:version>`
    * volume会把 `<HOST_PATH>`的内容映射到container里面的`<Container_PATH>` 当中.
  * 作用:
    * 对于需要不停修改的image， docker volume 让你不用重新修改DockerFile/build image, 只需要修改 docker volume/container里面的文件.
    * 使container的数据保存在 `var/lib/docker/volume` 下，即使container被删除，依旧有效，保持**数据持久化**.

* volume:
  * 使用方法:
    * `docker volume create <volume name>`- Create a volume.
    * `docker volume inspect <volume name>`- Inspect a volume.
    * `docker volume rm <volume name>` - Remove a volume. 
    * `docker run -v <DOCKER_VOL>:<CONTAINER_PATH> --name <name> <image name:version>`
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
## Build application with Docker
  * In DokcerFile:
    ```DockerFile
      FROM <language>:<version>  # choose base image.
      WORKDIR </Path>            # relativeDir 
      COPY . .                   # copy file from current dir to Docker daemon.
      RUN <command>              # execute any commands in a new layer on top of the current image and commit the results.
      CMD ["<name>"]             # provide defaults for an executing container
    ```
  * [12 factor apps](https://12factor.net/zh_cn/):
    * Codebase
      * 一份基准代码，多份部署.
    * Dependencies
      * 通过 **依赖清单** ，确切地声明所有依赖项.
    * Config
      * 应该用环境变量 `env` `env vars` 来储存应用配置，方便不同的部署做修改.
    * Backing services
      * 后端服务如数据库(**[MySQL](http://dev.mysql.com/)**, **[MongoDB](https://www.mongodb.com/)**)，消息队列(**[RabbitMQ](https://www.rabbitmq.com/)**)，缓存系统(**[Memcached](https://memcached.org/)**)作为附加资源.
      通过第三方调用(**AmazonS3**)或者API(**http://auth@api.twitter.com/**)访问来管理.
    * Build, release, run
    * Processes
    * Port binding
    * Concurrency
    * Disposability
      * 程序是可以**瞬间开启或者停止**，这有利于快速、弹性的伸缩应用，迅速部署变化的 代码 或 配置 ，稳健的部署应用.
    * Dev/prod parity
    * Logs
      * `ln -sf /dev/stdout <name>.log` 配合 shell script.
      * 通过输出流把log整合到一起发送给处理程序(**[logplex](https://github.com/heroku/logplex), [Fluentd](https://github.com/fluent/fluentd)**)，用于查看或者存档.
      * 把日志当成事件流(**stdout**) 发送到日志索引和分析系统(**[Splunk](https://www.splunk.com/)**)或者储存库(**[Hadoop/Hive](https://hive.apache.org/)**).
      * 通过以上方法可以有效减少硬盘储存.

    * Admin processes
## Docker Registry
* TODO
## Docker Networking
* TODO

## 常用命令
* `docker run` - 下载并执行容器.
* `docker start` - 开始已有容器.
* `docker stop` - 关闭已有容器.
* `docker pull` - 下载容器.
* `docker build <DockerFile>` - 建立容器.
* `docker exec <name> <path>` - 执行容器里的特定程序如bash.
* `docker ps` - 查看.

## 常用参数
* `-i` - run container in a interactive mode.
* `-t` - provide a tty for named container.
* `-d` - run in the background and return container ID.
