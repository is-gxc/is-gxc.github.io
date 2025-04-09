---
title: Docker基础
date: 2024-10-08 19:52:28
categories: Docker
tags: [Docker]
---

# Docker概念

![docker架构-来自docker官方](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/docker_arch.png)

Docker本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包好形成一个可交付的运行环境，这个打包好的运行环境就是image镜像文件。只有通过这个镜像文件才能生成Docker容器实例。

image文件可以看做是容器的模板。Docker根据image文件生成容器的实例。同一个image文件，可以生成多个同时运行的容器实例。

## 镜像文件

镜像是一个只读的模板。镜像可以用来创建 Docker 容器，一个镜像可以创建很多容器。本身也是一个文件，称之为镜像文件。

## 容器实例

一个容器运行一种服务，当我们需要的时候，就可以通过docker客户端创建一个对应的运行实例，也就是我们的容器

## 仓库

就是放一堆镜像的地方，我们可以把镜像发布到仓库中，需要的时候再从仓库中拉下来就可以了

## 整体框架

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/docker_process.png)

Docker是-个C/S模式的架构，后端是一个松耦合架构， 众多模块各司其职。
Docker运行的基本流程为:

1. 用户使用Docker Client与Docker Daemon建立通信，并发送请求给后者。
2. Docker Daemon作为Docker架构中的主体部分，首先提供Docker Server的功能使其可以接受Docker Client的请求。
3. Docker Engine执行Docker内部的一系列工作，每一项工作都是以一个Job的形式的存在。
4. Job的运行过程中，当需要容器镜像时，则从Docker Registry中下载镜像，并通过镜像管理驱动Graph driver将下载镜像以Graph的形式存储。
5. 当需要为Docker创建网络环境时，通过网络管理驱动Network driver创建并配置Docker容器网络环境。
6. 当需要限制Docker容器运行资源或执行用户指令等操作时，则通过Exec driver来完成。
7. Libcontainer是一项独立的容器管理包，Network driver以及Exec driver都是通过Libcontainer来实现具体对容器进行的操作。

# Docker安装

首先确认系统是 centos7及以上版本

## 卸载旧版本

``` shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

## yum安装gcc相关

Centos7 能上外网

yum -y install gcc

yum -y install gcc-c++



## 安装需要的软件包

```shell
sudo yum install -y yum-utils
```

## 设置stable镜像仓库

``` shell
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 更新yum软件包索引

``` shell
yum makecache fast
```

## 安装docker-ce

``` shell
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## 启动docker

``` shell
systemctl start docker
```

## 测试

``` shell
docker version
docker run hello-world
```

## 卸载

``` shell
systemctl stop docker
yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

## 获取阿里云镜像加速

1. 登录阿里云官网 -> 搜索容器镜像服务 -> 镜像工具 -> 镜像加速器

2. 获得加速器地址链接

3. 修改daemon配置文件/etc/docker/daemon.json来使用加速器

   ``` shell
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://xxxxx.mirror.aliyuncs.com"]
   }
   EOF
   ```

4. 重启服务

   ``` shell
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

   





# docker run执行过程

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/docker_run_process.png)

# 为什么Docker比虚拟机快

## 架构差异

虚拟机架构

``` shell
硬件
 └── 主机操作系统
      └── Hypervisor（如 VMware、VirtualBox、KVM）
           └── 每个虚拟机都有一套完整的操作系统
                └── 应用
```

Docker架构

``` shell
硬件
 └── 主机操作系统
      └── Docker 引擎（直接运行在主机 OS 上）
           └── 容器（共享主机内核）
                └── 应用
```



## 具体为什么更快

1. **无需启动完整 OS**
    容器只运行应用所需的环境和进程，而虚拟机要启动整个系统内核和系统服务。
2. **共享内核，减少系统调用开销**
    Docker 容器共享宿主机内核，因此系统调用不需要经过虚拟化层，性能更接近原生。
3. **资源利用率更高**
    容器是“轻量级”的，启动、销毁都非常快，适合弹性伸缩。
4. **更快的文件系统和网络访问**
    Docker 使用的是 Union FS（如 overlay2），读写效率相比虚拟硬盘文件快很多。

Docker 是“进程级”虚拟化，而虚拟机是“系统级”虚拟化。Docker 少了一个“虚拟整个系统”的层级，自然就快了。

# 常用命令

## 帮助启动类命令

- **启动docker：**systemctl start docker

- **停止docker：**systemctl stop docker

- **重启docker：**systemctl restart docker

- **查看docker状态：**systemctl status docker

- **开机启动：**systemctl enable docker

- **查看docker概要信息：**docker info

- **查看docker总体帮助文档：**docker --help

- **查看docker命令帮助文档：**docker 具体命令 --help

## 镜像命令

### docker images

**列出本地主机上的镜像**

各个选项说明:

REPOSITORY：表示镜像的仓库源

TAG：镜像的标签版本号

IMAGE ID：镜像ID

CREATED：镜像创建时间

SIZE：镜像大小 

同一仓库源可以有多个 TAG版本，代表这个仓库源的不同个版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像  

**options说明**

-a：列出本地所有的镜像（含历史映像层）

-q：只显示镜像ID



### docker search 某个xxx镜像名字

#### 网站

https://hub.docker.com

#### 命令

docker search [options] 镜像名字



**options说明：**

--limit : 只列出N个镜像，默认25个

docker search --limit 5 redis



### docker pull 某个xxx镜像名字

下载镜像
docker pull 镜像名字[:TAG]

没有TAG就是最新版 等价于 docker pull 镜像名字:latest



### docker ststem df 查看镜像/容器/数据卷所占空间

### docker rmi 某个xxx镜像名字ID

删除镜像
删除单个   docker rmi -f 镜像ID
删除多个   docker rmi -f 镜像名1:TAG 镜像名2:TAG
删除全部   docker rmi -f $(docker images -qa)



### 容器命令

#### 新建+启动容器

docker run [options] image [command]\[arg]

options说明
OPTIONS说明（常用）：有些是一个减号，有些是两个减号 
--name="容器新名字"       为容器指定一个名称；
-d: 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)；
-i：以交互模式运行容器，通常与 -t 同时使用；
-t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；也即启动交互式容器(前台有伪终端，等待交互)；
-P: 随机端口映射，大写P
-p: 指定端口映射，小写p

启动交互式容器（前台命令行）
docker run -it centos /bin/bash  参数说明：-i: 交互式操作。-t: 终端。centos : centos 镜像。/bin/bash：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。要退出终端，直接输入 exit: 

### 列出当前所有正在运行的容器

docker ps [options] 

options说明
OPTIONS说明（常用）： 
-a :列出当前所有正在运行的容器+历史上运行过的
-l :显示最近创建的容器。
-n：显示最近n个创建的容器。
-q :静默模式，只显示容器编号。



### 退出容器

exit   run进去的容器，exit退出，容器停止
ctrl+p+q   run进去容器，ctrl+p+q退出，容器不停止

### 启动已经停止运行的容器

docker start 容器ID或容器名

### 停止容器

docker stop 容器ID或者容器名

### 强制停止容器

docker kill 容器ID或容器名

## 删除已停止的容器

docker rm 容器ID

一次性删除多个容器实例
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm

### 重要命令

#### 启动守护式容器（后台服务器）

在大部分的场景下，我们希望 docker 的服务是在后台运行的，
我们可以过 -d 指定容器的后台运行模式。

docker run -d 容器名
使用镜像centos:latest以后台模式启动一个容器docker run -d centos 问题：然后docker ps -a 进行查看, 会发现容器已经退出很重要的要说明的一点: Docker容器后台运行,就必须有一个前台进程.容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。 这个是docker的机制问题,比如你的web容器,我们以nginx为例，正常情况下,我们配置启动服务只需要启动响应的service即可。例如service nginx start但是,这样做,nginx为后台进程模式运行,就导致docker前台没有运行的应用,这样的容器后台启动后,会立即自杀因为他觉得他没事可做了.所以，最佳的解决方案是,将你要运行的程序以前台进程的形式运行，常见就是命令行模式，表示我还有交互操作，别中断，O(∩_∩)O哈哈~

启动演示：
前台交互式启动  docker run -it redis:6.0.8
后台守护式启动  docker run -d redis:6.0.8

#### 查看容器日志

docker logs 容器ID

#### 查看容器内运行的进程

docker top 容器ID

#### 查看容器内部细节

docker inspect 容器ID

#### 进入正在运行的容器并以命令行进行交互

docker exec -it 容器ID bashShell
重新进入docker attach 容器ID

上述两个区别：
attach 直接进入容器启动命令的终端，不会启动新的进程
用exit退出，会导致容器的停止。
exec 是在容器中打开新的终端，并且可以启动新的进程
用exit退出，不会导致容器的停止。

推荐大家使用 docker exec 命令，因为退出容器终端，不会导致容器的停止。

#### 从容器内拷贝文件到主机上

docker cp  容器ID:容器内路径 目的主机路径

#### 导入和导出容器

export 导出容器的内容留作为一个tar归档文件[对应import命令]
import 从tar包中的内容创建一个新的文件系统再导入为镜像[对应export]

docker export 容器ID > 文件名.tar
cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号 



# 镜像

## 镜像是什么

是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境（包括代码、运行时需要的库、环境变量和配置文件等），这个打包好的运行环境就是image镜像文件。

只有通过这个镜像文件才能生成Dokcer容器实例（类似java种new出来一个对象）



## 联合文件系统

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层一层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下（unite several directories into a single virtual filesystem）。Union文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。
特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录



## docker镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载botfs文件系统，在Docker镜像的最底层是引导文件系统bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system)，在bootfs之 上。 包含的就是典型Linux系统中的/dev, /proc, /bin, /etc等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu, Centos等等。

平时我们安装进虚拟机的CentOS都是好几个G，为什么docker这里才200M？？对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。

## 为什么docker镜像要采用这种分层结构呢？

镜像分层最大的一个好处就是共享资源，方便复制迁移，就是为了复用。 比如说有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。 

## 容器层

Docker镜像层都是只读的，容器层是可写的

当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中。

只有容器层是可写的，容器层下面的所有镜像层都是只读的。

## docker镜像commit操作案例

docker commit提交容器副本使之成为一个新的镜像

docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]

## 虚悬镜像

**虚悬镜像（Dangling Image）是Docker中一种特殊的镜像状态，它指的是仓库名（镜像名）和标签（TAG）都是`<none>`的镜像。** 这些镜像通常是由于以下原因产生的：

1. **构建过程中的错误或取消**：在构建镜像的过程中，如果由于某些原因（如构建取消或构建失败）导致镜像没有被正确命名或标记，就会形成虚悬镜像。
2. **删除标签**：如果手动删除了一个镜像的标签，而这个镜像没有被其他标签引用，那么它也会变成虚悬镜像。
3. **镜像分层**：Docker镜像是由多个分层组成的，当某个分层不再被任何镜像引用时，它也会变成虚悬镜像。

虚悬镜像的特点包括：

- **未被任何容器引用**：虚悬镜像没有被任何正在运行的或停止的容器所引用。
- **占用磁盘空间**：尽管虚悬镜像没有实际用途，但它们仍然会占用磁盘空间。
- **不会自动清理**：Docker的清理工具不会自动删除虚悬镜像，需要手动进行清理。

为了管理虚悬镜像，可以采取以下措施：

- **定期清理**：使用`docker image prune`或`docker system prune`命令来清理虚悬镜像和其他无用资源。
- **手动删除**：使用`docker images -f "dangling=true"`命令列出所有虚悬镜像，然后使用`docker rmi <image_id>`命令删除指定的镜像。
- **预防产生**：在构建镜像时，为镜像打上合适的标签，避免使用`latest`标签，以减少虚悬镜像的产生。同时，合理使用多阶段构建等技巧，也可以减少中间镜像的产生。



## 总结

ocker中的镜像分层，支持通过扩展现有镜像，创建新的镜像。类似Java继承于一个Base基础类，自己再按需扩展。新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层



# 容器数据卷

容器数据卷记得加入   **--privileged=true**

Docker挂载主机目录访问如果出现cannot open directory .: Permission denied解决办法：在挂载目录后多加一个--privileged=true参数即可 如果是CentOS7安全模块会比之前系统版本加强，不安全的会先禁止，所以目录挂载的情况被默认为不安全的行为，在SELinux里面挂载目录被禁止掉了，如果要开启，我们一般使用--privileged=true命令，扩大容器的权限解决挂载目录没有权限的问题，也即使用该参数，container内的root拥有真正的root权限，否则，container内的root只是外部的一个普通用户权限。

## 容器数据卷是什么

类似我们Redis里面的rdb和aof文件

将docker容器内的数据保存进宿主机的磁盘中

运行一个带有容器卷存储功能的容器实例： docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录      镜像名

## 作用

我们对数据的要求希望是持久化的 Docker容器产生的数据，如果不备份，那么当容器实例删除后，容器内的数据自然也就没有了。为了能保存数据在docker中我们使用卷。 

**特点：**

1：数据卷可在容器之间共享或重用数据

2：卷中的更改可以直接实时生效

3：数据卷中的更改不会包含在镜像的更新中

4：数据卷的生命周期一直持续到没有容器使用它为止 



# DockerFile

## 概述

Dockerfile是用来构建Docker镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本。

## DockerFile构建过程解析

1. 每条保留字都必须为大写字母且后边要跟随至少一个参数
2. 指令按照从上到下，顺序执行
3. #表示注释
4. 每条指令都会创建一个新的镜像层并对镜像进行提交

### Docker执行DockerFile的大致流程

1. docker从基础镜像运行一个容器
2. 执行一条命令对容器做出修改
3. 执行类似docker commit的操作提交一个新的镜像层
4. docker再基于刚提交的镜像运行一个新容器
5. 执行dockerfile中的下一条指令直到所有指令都执行完成

---

从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，  Dockerfile是软件的原材料 、Docker镜像是软件的交付品、Docker容器则可以认为是软件镜像的运行态，也即依照镜像运行的容器实例。

Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

1. Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等; 
2. Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时会真正开始提供服务;
3. Docker容器，容器是直接提供服务的。  

## DockerFile常用保留字指令

**FROM** 
基础镜像，当前新镜像是基于哪个镜像的，指定一个已经存在的镜像作为模板，第一条必须是from

**MAINTAINER**
镜像维护者的姓名和邮箱地址

**RUN**
容器构建时需要运行的命令
两种格式：shell格式（ RUN yum -y install vim）；exec格式。
RUN是在 docker build时运行

**EXPOSE**
当前容器对外暴露出的端口

**WORKDIR**
指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点

**USER**
指定该镜像以什么样的用户去执行，如果都不指定，默认是root

**ENV**
用来在构建镜像过程中设置环境变量

ENV MY_PATH /usr/mytest这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一样；也可以在其它指令中直接使用这些环境变量， 比如：WORKDIR $MY_PATH

**ADD**
将宿主机目录下的文件拷贝进镜像且会自动处理URL和解压tar压缩包

**COPY**
类似ADD，拷贝文件和目录到镜像中。

将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置
COPY src dest
COPY ["src", "dest"]
<src源路径>：源文件或者源目录
<dest目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

**VOLUME**
容器数据卷，用于数据保存和持久化工作

**CMD**
指定容器启动后的要干的事情

**注意**
Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换
它和前面RUN命令的区别：CMD是在docker run 时运行。RUN是在 docker build时运行。



**ENTRYPOINT**
类似于 CMD 指令，但是ENTRYPOINT不会被docker run后面的命令覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。

ENTRYPOINT可以和CMD一起用，一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参。当指定了ENTRYPOINT后，CMD的含义就发生了变化，不再是直接运行其命令而是将CMD的内容作为参数传递给ENTRYPOINT指令。

案例：

``` shell
FROM nginx
ENTRYPOINT ["nginx", "-c"] #定参
CMD ["/etc/nginx/nginx.conf"] #变参
```

| 是否传参         | 按照dockerfile编写执行         | 传参运行                                     |
| ---------------- | ------------------------------ | -------------------------------------------- |
| docker命令       | docker run nginx:test          | docker run nginx:test -c /etc/nginx/new.conf |
| 衍生出的实际命令 | nginx -c /etc/nginx/nginx.conf | nginx -c /etc/nginx/new.conf                 |



在run的时候，在docker run后边传了新的参数，会覆盖 nginx.conf变成new.conf

**优点**在执行docker run的时候可以指定 ENTRYPOINT 运行所需的参数。

**注意**如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。

## 案例

自定义镜像mycentosjava8

要求

Centos7镜像具备vim+ifconfig+jdk8

### 编写

``` shell
FROM centos7
#安装vim编辑器
RUN yum -y install vim
#安装ifconfig命令查看网络
IPRUN yum -y install net-tools
#安装java8及lib库
RUN yum -y install glibc.i686RUN mkdir /usr/local/java
#ADD 是相对路径jar,把jdk-8u171-linux-x64.tar.gz添加到容器中,安装包必须要和Dockerfile文件在同一位置
ADD jdk-8u171-linux-x64.tar.gz /usr/local/java/
#配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_171
ENV JRE_HOME $JAVA_HOME/jreENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATHENV PATH $JAVA_HOME/bin:$PATH EXPOSE 80 
CMD echo $MYPATHCMD echo "success--------------ok"CMD /bin/bash 
```

### 构建

docker build -t 新镜像名字:TAG .  （docker build -t centosjava8:1.5 .  ）

注意：最后有个点



### 运行

docker run -it 新镜像名字:TAG 

docker run -it centosjava8:1.5 /bin/bash

此处再体会一下联合文件系统

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录





# Docker网络

docker不启动，默认网络情况：ens33 lo virbr0

docker启动后，网络情况：

会产生一个名为docker0的虚拟网桥

## 作用

容器间的互联和通信以及端口映射

容器IP变动时候可以通过服务名直接网络通信而不受到影响

## 常用基本命令

- **查看网络** docker network ls

- **查看网络源数据** docker network inspect  XXX网络名字

- **删除网络** docker network rm XXX网络名字

- **删除没有用的网络** docker network prune

- **创建网络** docker network create

## 网络模式

### 总体介绍

bridge模式：使用--network  bridge指定，默认使用docker0

host模式：使用--network host指定

none模式：使用--network none指定

container模式：使用--network container:NAME或者容器ID指定

## bridge

Docker 服务默认会创建一个 docker0 网桥（其上有一个 docker0 内部接口），该桥接网络的名称为docker0，它在内核层连通了其他的物理或虚拟网卡，这就将所有容器和本地主机都放到同一个物理网络。

Docker 默认指定了 docker0 接口 的 IP 地址和子网掩码，让主机和容器之间可以通过网桥相互通信。 

查看 bridge 网络的详细信息，并通过 grep 获取名称项docker network inspect bridge | grep name ifconfig

### 说明

1 Docker使用Linux桥接，在宿主机虚拟一个Docker容器网桥(docker0)，Docker启动一个容器时会根据Docker网桥的网段分配给容器一个IP地址，称为Container-IP，同时Docker网桥是每个容器的默认网关。因为在同一宿主机内的容器都接入同一个网桥，这样容器之间就能够通过容器的Container-IP直接通信。 

2 docker run 的时候，没有指定network的话默认使用的网桥模式就是bridge，使用的就是docker0。在宿主机ifconfig,就可以看到docker0和自己create的network(后面讲)eth0，eth1，eth2……代表网卡一，网卡二，网卡三……，lo代表127.0.0.1，即localhost，inet addr用来表示网卡的IP地址 

3 网桥docker0创建一对对等虚拟设备接口一个叫veth，另一个叫eth0，成对匹配。  

3.1 整个宿主机的网桥模式都是docker0，类似一个交换机有一堆接口，每个接口叫veth，在本地主机和容器内分别创建一个虚拟接口，并让他们彼此联通（这样一对接口叫veth pair）；  

3.2 每个容器实例内部也有一块网卡，每个接口叫eth0；  3.3 docker0上面的每个veth匹配某个容器实例内部的eth0，两两配对，一一匹配。 

通过上述，将宿主机上的所有容器都连接到这个内部网络上，两个容器在同一个网络下,会从这个网关下各自拿到分配的ip，此时两个容器的网络是互通的。

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/docker_net_bridge.png)

## host

直接使用宿主机的 IP 地址与外界进行通信，不再需要额外进行NAT 转换。

### 说明

容器将不会获得一个独立的Network Namespace， 而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡而是使用宿主机的IP和端口。

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/docker_net_host.png)

## none

在none模式下，并不为Docker容器进行任何网络配置。 也就是说，这个Docker容器没有网卡、IP、路由等信息，只有一个lo需要我们自己为Docker容器添加网卡、配置IP等。

禁用网络功能，只有lo标识(就是127.0.0.1表示本地回环)

## container

container⽹络模式 新建的容器和已经存在的一个容器共享一个网络ip配置而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。

![](https://gxc-hexo-blog.oss-cn-beijing.aliyuncs.com/blog/docker_net_container.png)

## 自定义网络

在用不用自定义网络之前，不同容器之间使用ping ip是可以通的，但是ping 服务名是不通的，但是在容器里ip是会发生变化的，不是唯一标识，有可能下次会有别的容器来使用当前ip，此时就会有问题。当使用了自定义网络，同一网络下的，可以使用 ping 服务名 是通的，所以可以作为 网络中的唯一标识。

**自定义网络本身就维护好了主机名和ip的对应关系（ip和域名都能通）**
