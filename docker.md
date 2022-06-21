# 前言

## 什么是docker

[docker官网](https://www.docker.com)

[docker中文学习网站](https://docker_practice.gitee.io/zh-cn/)

### 官方说明：

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210901110332047.png)

我们为你准备了完美的容器的解决方案，无论你是谁，不管你在你的容器之旅中的何处。

### **官方定义**

`docker`是一个容器技术

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210901110712980.png)

#### **docker概述**

`Docker` 是一个用于开发、传送和运行应用程序的开放平台。`Docker` 能够将应用程序与基础设施分开，以便可以快速交付软件。使用 `Docker`，可以像管理应用程序一样管理基础设施。通过利用 `Docker` 的快速交付、测试和部署代码的方法，可以显着减少编写代码和在生产中运行代码之间的延迟。

#### **docker平台**

`Docker` 提供了在称为容器的松散隔离环境中打包和运行应用程序的能力。隔离和安全性允许您在给定主机上同时运行多个容器。容器是轻量级的，包含运行应用程序所需的一切，因此您无需依赖主机上当前安装的内容。您可以在工作时轻松共享容器，并确保与您共享的每个人都获得以相同方式工作的相同容器。

`Docker` 提供工具和平台来管理容器的生命周期：

> 使用容器开发您的应用程序及其支持组件。
>
> 容器成为分发和测试应用程序的单元。
>
> 准备就绪后，将应用程序作为容器或编排服务部署到生产环境中。无论您的生产环境是本地数据中心、云提供商还是两者的混合，这都是一样的。

### 我对docker的理解：

我理解其为一个大虚拟机里面的小虚拟机（当然`docker`和虚拟机之间还是有区别的，可以这么理解但是不可以这么认为）。所谓`docker`容器就是将应用程序以及其需要的环境打包放在一起，其单独拿出来其实就相当于一个你自己机器上的虚拟机的复制版。而且取出使用很方便，比较轻量，对于大型团队的开发省去很多麻烦。**【待补充】。。。**

### docker logo说明：

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210901112129103.png)

**鲸鱼**：代表的是`docker`引擎，这意味着想要使用`docker`就需要安装`docker`引擎

**鲸鱼身上的箱子**：这代表的是一个一个的软件环境，也就是容器。这意味着在`dockers`引擎上可以同时平稳的运行多个容器。

## 为什么用docker

`docker`将程序以及使用环境直接打包在一起，无论在哪个机器上，保证了环境的一致。这样就不用担心自己写的程序可以在自己机器上跑，去了别的机器就报各种让人烦死掉的错误。

> 一致的运行环境，更轻松的迁移

服务器自己的程序挂了，结果发现是别人的程序出了问题把内存吃完了，自己的程序因为内存不够挂了。因为公司不可能给每个人配一台机器，因此更多的时候是很多人公用一个服务器，自己的程序就会比较有可能收到别人的影响。

> 对进程进行封装隔离，容器与容器之间互不影响，更高效的利用系统资源

假如公司要举办活动，会有巨大的流量进来，公司需要多部署几十台上百台服务器。如果用传统的运维方式，在每一台新服务器上安装各种环境，还要对应的版本，可能会折磨的运维人员死掉。

> 通过镜像复制N多个环境一致的容器

## docker与虚拟机的对比

### 虚拟机

虚拟机运行软件环境之前必须要携带操作系统，也正是因为操作系统，使得本身很小的应用程序变得非常大，很笨重。

通过虚拟机在资源调度上经过很多步骤，另外在调用宿主机的`CPU`、磁盘等资源的时候，以内存举例：虚拟机是利用`Hypervisor`去虚拟化内存，整个调用过程是`虚拟内存->虚拟物理内存->真正物理内存`

### docker

而`docker`是不携带操作系统的，因此`docker`的应用非常的轻巧。

`docker`在资源调度上也轻便许多，同样以内存为例，`docker`的整个调用过程是`虚拟内存->真正物理内存`，相比较虚拟机省去了一个步骤，调用内存的速度显然更快。

### 二者各方面对比

|             | 传统虚拟机                           | docker容器                          |
| ----------- | ------------------------------------ | ----------------------------------- |
| 磁盘占用    | 几个G到几十个G左右                   | 几十M到几百M左右                    |
| 启动速度    | （开机到运行项目）几分钟             | （开启容器到运行项目）几秒          |
| CPU内存占用 | 虚拟机操作系统非常占用CPU和内存      | docker引擎占用极低                  |
| 安装管理    | 需要专门的运维技术                   | 安装、管理方便                      |
| 应用部署    | 每次部署都很费时费力                 | 从第二次部署开始轻松便捷            |
| 耦合性      | 多个应用服务安装到一起，容易互相影响 | 每个应用服务一个容器，达成隔离      |
| 系统依赖    | 无                                   | 需求相同或相似的内核，目前推荐linux |

## docker的安装

### bash安装（所有平台通用）

在测试或者开发环境中`Docker`官方为了简化安装流程，提供了一套便捷的安装脚本，`CentOS`系统上可以使用这套脚本安装，另外可以通过`--mirror`选项使用国内源进行安装：执行完这个命令后，脚本就会自动将一切准备工作做好，并且把`Docker`的稳定（`stable`）版本安装在系统中

```
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```

启动`docker`

```
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

创建`docker`用户组

```
$ sudo groupadd docker 
```

将当前用户加入`docker`组

```
sudo usermod -aG docker $USER
```

测试`docker`安装是否正确

```
$ docker run hello-world
```

### 在windows系统中安装

**下载**：[官方下载地址](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe)

**安装**：直接打开`exe`文件安装即可

**注意事项**：在`win10`系统中安装`docker`需要打开`Hyper-v`，具体过程如下：

> 打开控制面板 ---> 程序和功能 ---> 启用和关闭windous功能 ---> 勾选Hyper-v ---> 完场后重启

## docker架构

### docker核心架构

在了解`docker`的架构之前，我们先查看一下`docker`的信息，使用命令`docker info`查看（以下只截取一部分内容）

```go
Client:
 Context:    default
 Debug Mode: false

Server:
 Containers: 6
  Running: 0
  Paused: 0
  Stopped: 6
 Images: 5
 Server Version: 20.10.7
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
```

可以发现，`docker`是有`Client`端和`Server`端，可见`docker`是一个很典型的**CS架构**。这意味着我们要使用`docker`客户端执行命令的时候，必须要开启`docker`的服务端。另外，`CS`架构意味着一个服务端可以对应多个客户端，并且不局限于命令行的模式，在日后对`docker`了解更加深入的时候，可以使用`idea`等辅助工具，这样使用`docker`起来会更加的方便。

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210901183120445.png)

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210906171924851.png)

### docker中的核心概念

#### 镜像		images

**定义**：一个镜像即代表一个软件	如：`mysql`镜像	`redis`镜像	`nacos`镜像	`nginx`镜像等....

**特点**：只读

#### 容器		container

**定义**：基于某个镜像运行一次就是生成一个程序实例，一个程序实例也称之为一个容器

**特点**：可读可写

#### 仓库		repository

**定义**：存储`docker`中的所有镜像的具体位置

**远程仓库**：`docker`在世界范围维护的一个唯一远程仓库，存储着全世界开发者镜像和官方镜像

**本地仓库**：当前自己机器中下载的镜像存储的位置，只存放自己使用过的镜像以及自定义镜像

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210901172754272.png)

## docker 下载镜像的加速设置

### 背景

由于`docker`的远程仓库位处国外，由于国内外的网络关系，直接从`docker`的远程仓库拉取镜像是很慢的，基于此背景下，阿里巴巴在国内将`docker`的远程仓库复制到自己的服务器，建立国内的`docker`远程仓库，开源并进行维护，与官方的`docker`远程仓库里面的镜像是一样的。（阿里牛逼）

### 设置阿里云加速

> 进入阿里云官网
>
> 登录阿里云账号
>
> 在产品服务中搜索“容器服务”，找到“容器镜像服务”
>
> 在打开的页面中寻找到镜像加速器服务
>
> 切换使用系统
>
> 按照流程设置镜像加速服务
>
> 检查是否安装成功：docker info，查看是否有Registry Mirrors这一栏即可。

### 我的阿里云加速器

#### ubuntu镜像加速配置

针对`Docker`客户端版本大于 `1.10.0` 的用户

通过修改`daemon`配置文件`/etc/docker/daemon.json`来使用加速器

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xl8jcf39.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### centos镜像加速配置

针对`Docker`客户端版本大于 `1.10.0` 的用户

通过修改`daemon`配置文件`/etc/docker/daemon.json`来使用加速器

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xl8jcf39.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### mac镜像加速配置

1. 安装／升级`Docker`客户端

对于`10.10.3`以下的用户 推荐使用`Docker Toolbox`

`Mac`安装文件：http://mirrors.aliyun.com/docker-toolbox/mac/docker-toolbox/

对于`10.10.3`以上的用户 推荐使用`Docker for Mac`

`Mac`安装文件：http://mirrors.aliyun.com/docker-toolbox/mac/docker-for-mac/

2. 配置镜像加速器

针对安装了`Docker Toolbox`的用户，可以参考以下配置步骤：

创建一台安装有`Docker`环境的`Linux`虚拟机，指定机器名称为`default`，同时配置`Docker`加速器地址。

```
docker-machine create --engine-registry-mirror=https://xl8jcf39.mirror.aliyuncs.com -d virtualbox default
```

查看机器的环境配置，并配置到本地，并通过`Docker`客户端访问`Docker`服务。

```
docker-machine env defaulteval "$(docker-machine env default)"docker info
```

针对安装了`Docker for Mac`的用户，您可以参考以下配置步骤：

在任务栏点击 `Docker Desktop` 应用图标 -> `Perferences`，在左侧导航菜单选择 `Docker Engine`，在右侧输入栏编辑 `json` 文件。将[镜像加速器地址](https://xl8jcf39.mirror.aliyuncs.com)加到`"registry-mirrors"`的数组里，点击 `Apply & Restart`按钮，等待`Docker`重启并应用配置的镜像加速器。

#### windows镜像加速配置

##### 1. 安装／升级Docker客户端

对于`Windows 10`以下的用户，推荐使用`Docker Toolbox`

`Windows`安装文件：http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/

对于`Windows 10`以上的用户 推荐使用`Docker for Windows`

`Windows`安装文件：http://mirrors.aliyun.com/docker-toolbox/windows/docker-for-windows/

##### 2. 配置镜像加速器

针对安装了`Docker Toolbox`的用户，您可以参考以下配置步骤：

创建一台安装有`Docker`环境的`Linux`虚拟机，指定机器名称为`default`，同时配置`Docker`加速器地址。

```
docker-machine create --engine-registry-mirror=https://xl8jcf39.mirror.aliyuncs.com -d virtualbox default
```

查看机器的环境配置，并配置到本地，并通过`Docker`客户端访问`Docker`服务。

```
docker-machine env defaulteval "$(docker-machine env default)"docker info
```

针对安装了`Docker for Windows`的用户，您可以参考以下配置步骤：

在系统右下角托盘图标内右键菜单选择 `Settings`，打开配置窗口后左侧导航菜单选择 `Docker Daemon`。编辑窗口内的`JSON`串，填写下方加速器地址：

```
{
  "registry-mirrors": ["https://xl8jcf39.mirror.aliyuncs.com"]
}
```

编辑完成后点击 `Apply` 保存按钮，等待`Docker`重启并应用配置的镜像加速器。

#### 注意

`Docker for Windows` 和 `Docker Toolbox`互不兼容，如果同时安装两者的话，需要使用`hyperv`的参数启动。

```
docker-machine create --engine-registry-mirror=https://xl8jcf39.mirror.aliyuncs.com -d hyperv default
```

`Docker for Windows` 有两种运行模式，一种运行`Windows`相关容器，一种运行传统的`Linux`容器。同一时间只能选择一种模式运行。

# docker入门

## docker的第一个程序

官方发布的入门程序：

```
sudo docker run hello-world:latest
```

```go
gdcgx@gdcgx-virtual-machine:~$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
b8dfde127a29: Pull complete 
Digest: sha256:7d91b69e04a9029b99f3585aaaccae2baa80bcf318f4a5d2165a9898cd2dc0a1
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

### 为何输出是这样的？

`docker run`是根据一个镜像运行容器，可以看到命令中run的是`hello-world`这个镜像，没有指定版本，因此默认的是最新的`hello-world`镜像版本。由于在此前我并没有使用过`hello-world`镜像，因此我的本地镜像是没有它的，所以输出结果第二行说明 `没有在本地找到hello-world镜像` 。既然没有镜像，那`docker`就会自动在远程仓库给我拉取最新版本的`hello-world`镜像，如输出结果第三行所示。在远程仓库拉取镜像完毕之后，就会自动运行容器，并输出了容器执行的结果啦。

## docker运行流程

根据上述入门程序，可以总结得出docker run的一个运行流程。

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210901181808293.png)

## docker常用命令

### dockers引擎以及帮助命令

```go
docker info     //展示docker信息（版本、状态等内容）
docker version  //展示docker版本号信息
docker --help || docker  //查看docker所有帮助命令
```

### 操作镜像 images 相关命令

```GO
#1.查看本机中所有镜像
docker images    //列出本地仓库储存的所有镜像  
	 		 -a //列出所有镜像（包括中间映像层）
	 		 -q //只显示镜像id
	 		 
#2.搜索镜像
docker search [options] //去docker hub上查询当前镜像（可指定版本号，但不建议）
			 -s 指定值   //列出收藏数不少于指定值的镜像
			 --no-trunc  //显示完整的镜像信息
			 --limit int //限制列出个数
			 
#3.从仓库中下载镜像
docker pull 镜像名：版本号				//下载镜像
//docker pull 镜像名：@GIGEST（摘要） 	//下载镜像

#4.删除镜像
docker rmi [镜像id]||[镜像名：镜像版本]	    //删除镜像
		      -f	  					 //强制删除
docker image rm [镜像id]||[镜像名：镜像版本]  //删除镜像
			  -f     					 //强制删除

#5.将打包的tar镜像文件导入自己的docker本地仓库
docker load -i [导入tar的镜像文件名]	//需要在tar文件所在文件夹下操作，不然要指定路径

#6.对镜像重命名
重命名：docker tag IMAGEID(镜像id) REPOSITORY:TAG(仓库：标签)
```

**补充**：

**docker images**

`REPOSITORY`（镜像名称）		`TAG`（版本）		`IMAGE ID`（id）		`CREATED`（创建时间）		`SIZE`（镜像大小）

**docker pull 镜像名：@GIGEST（摘要）**

如何获取摘要？

在`docker hub`中搜索需要的镜像，找到其并打开，如下图位置就是摘要。这种方式比起第一种属实有点蠢，**才用，仅此记录而已。

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210901185514334.png)

### 操作容器 contrainer 容器命令

```go
#1.运行容器
docker run [镜像名]    //新建并启动容器
		   --name	 //为容器取别名
		   -d		 //在后台启动容器
		   -p 映射端口号：原始端口号	//指定端口号启动（可列多个）：由于容器自身是隔离的，所以外部是不能访问容器内ip端口的，而我们自然是希望能访问到这个端口的，因此产生了-p，将容器内的原始端口映射到宿主机上的映射端口上，将两个端口绑定，这样的话外部访问宿主机映射端口时，就可以访问到容器内的原始端口。
ex： docker run -it --name myUbuntu -p 8888:8000 ubuntu：20.04
	docker -d --name myUbuntu -P ubuntu
	
#2.查看运行的容器
docker ps 		//列出所有正在运行的容器
		-a	    //列出所有的容器，包括关闭的
		-q		//静默模式，只显示容器id 
		-qa 	//返回所有容器id
		
#3.停止|关闭|重启容器
docker start [容器名]||[容器id]		//开启容器
docker restart [容器名]||[容器id]	//重启容器
docker stop [容器名]||[容器id]		//正常停止容器运行
docker kill [容器名]||[容器id] 		//立即停止容器运行

#4.删除容器
docker rm -f [容器名]||[容器id]		//删除指定容器
docker rm -f $(docker ps -aq)		//删除所有容器

#5.查看容器内进程
docker roo [容器名]||[容器id]		//查看容器内的进程

#6.查看容器内部细节
docker inspect [容器id]			  //查看容器内部细节

#7.查看容器的运行日志
docker logs [容器名]||[容器id]		//查看容器日志
			-t					  //加入时间戳
			-f					  //跟随最新的日志打印
			-tail int			   //显示最后多少条
			
#8.进入容器内部
docker exec [容器名]||[容器id] bash	//进入容器执行命令
			-i					  //以交互模式运行容器，通常与-t一起使用
			-t					  //分配一个伪终端，shell窗口，bash
交互模式：通过学习，可以知道容器是操作系统进程间的隔离，因此可以理解城一个容器就是一个最精简的操作系统，只有最基本的命令例如cd、ls、echo等，没有进阶的命令例如ll、vi、vim等。-it就相当于打开这个最精简操作系统的终端

#9.容器和宿主机之间复制文件
docker cp 文件|目录 [容器名]||[容器id]	//将宿主机文件复制到容器内部
docker cp [容器id/name：容器内资源路径（文件/目录）] 宿主机目录路径	//将容器内资源拷贝到主机上

#10.数据卷（volum）实现与宿主机共享目录
docker run -v 宿主机路径|任意别名：/容器内路径 镜像名	//自定义数据卷目录（启动容器时作为docker run参数开启数据卷）
docker run -v 宿主机路径|任意别名：/容器内路径：ro 镜像名 //自定义数据卷目录，容器对目录只读（即宿主机可以对数据卷目录进行修改，容器不可以）
docker run -d -p 80：80 --name ubuntu -v aa:[容器目录] Ubuntu：20.04    //自动数据卷目录。aa代表一个数据卷名字，可以随便写，docker不存在时自动创建这个数据卷同时自动映射宿主机中某个目录。同时在启动容器时会将aa对应容器目录中全部内容复制到aa映射目录中。

#10.将容器打包成一个新的镜像
docker commit -m "描述信息" -a "作者信息" [容器id/容器name] [打包的镜像名称]：[标签/版本]

#11.将镜像备份出来
docker save [镜像名称] -o [文件名]
```

**补充**：

**自定义数据卷**：这种方式更适合当我们需要主动去对容器内数据做修改的时候，也就是“写”操作较多的时候。

**自动数据卷**：这种方式更适合当我们需要读取容器内部数据，需要保护容器内数据的时候，也就是“读”操作较多的时候。因为这种方式不会在我们误操作删除容器后，导致容器中挂载的项目的运行记录以及生成的数据全部被删除。

# docker的镜像原理

## 镜像是什么？

> 镜像是一种轻量级的，可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需要的内容，包括代码，运行时所需要的库，环境变量和配置文件。

## 为什么一个镜像会很大？

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210902180455715.png)

> 镜像就像是一个花卷，一层一层的组成

**UnionFS**（联合文件系统），`docker`的镜像设计原理：

`Union`文件系统是一种分层，轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下。`Union`文件系统是`Docker`镜像的基础。这种文件系统特性就是一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录。

### 镜像这么大的原因

一个镜像需要包括三部分，不仅仅是一个软件包那么简单，其由软件包所需的操作系统依赖，软件自身依赖，以及自身软件包三个部分组成。

## docker镜像原理

`docker`的镜像实际是由一层一层的文件系统组成。

> `bootfs` (`boot fle system`）主要包含`bootloader`和`kernel`，`bootloader`主要是引导加载`kernel`,，`Linux`刚启动时会加载`bootfs`文件系统。在`docker`镜像的最底层就是`bootfs`。这一层与`Linux/Unix`系统是一样的，包含`boot`加载器(`bootoader`)和内核(`kernel`)。当`boot`加载完,后整个内核就都在内存中了，此时内存的使用权已由`bootfs`转交给内核，此时会卸载`bootfs`。

> `rootfs` (`root file system`)，在`bootfs`之上，包含的就是典型的`linux`系统中的`/dev，/proc，/bin ，/etc`等标准的目录和文件。`rootfs`就是各种不同的操作系统发行版，比如`ubuntu/CentOS`等等。

> 我们平时安装进虚拟机的`centos`都有1到几个GB，为什么`docker`这里才`200MB`?对于一个精简的`OS`,`rootfs`可以很小，只需要包括最基本的命令，工具，和程序库就可以了，因为底层直接使用`Host`的`Kernal`，自己只需要提供`rootfs`就行了。由此可见不同的`linux`发行版，他们的`bootfs`是一致的，`rootfs`会有差别。因此不同的发行版可以共用`bootfs`。

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210902182707863.png)

## 为什么docker镜像采用分层镜像原理？

> 最大的一个好处就是资源共享，采用分层机制实现基础层共享，从而减小`docker`仓库整体体积

比如:有多个镜像都是从相同的`base`镜像构建而来的，那么宿主机只需在磁盘中保存一份`base`镜像。同时内存中也只需要加载一份`base`镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。`Docker`镜像都是只读的。当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称为容器层，容器层之下都叫镜像层。

# docker高级网络配置

## docker容器与外界通信机制——桥接模式

当 `Docker` 启动时，会自动在主机上创建一个 `docker0` 虚拟网桥，实际上是 `Linux` 的一个 `bridge`，可以理解为一个软件交换机。它会在挂载到它的网口之间进行转发。

同时，`Docker` 随机分配一个本地未占用的私有网段（在 [RFC1918 (opens new window)](https://datatracker.ietf.org/doc/html/rfc1918)中定义）中的一个地址给 `docker0` 接口。比如典型的 `172.17.42.1`，掩码为 `255.255.0.0`。此后启动的容器内的网口也会自动分配一个同一网段（`172.17.0.0/16`）的地址。

当创建一个 `Docker` 容器的时候，同时会创建了一对 `veth pair` 接口（当数据包发送到一个接口时，另外一个接口也可以收到相同的数据包）。这对接口一端在容器内，即 `eth0`；另一端在本地并被挂载到 `docker0` 网桥，名称以 `veth` 开头（例如 `vethAQI2QT`）。通过这种方式，主机可以跟容器通信，容器之间也可以相互通信。`Docker` 就创建了在主机和所有容器之间一个虚拟共享网络。

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/network.6ad909f2.png)

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210906150703353.png)

接下来的部分将介绍在一些场景中，`Docker` 所有的网络定制配置。以及通过 `Linux` 命令来调整、补充、甚至替换 `Docker` 默认的网络配置。

### 网桥机制的缺陷

由于`docker`启动时，默认会为所有开启的容器分配一个网桥的网段，这就导致了一个问题——所有容器共用一个网桥网段，当某个容器频繁与外界进行通信时，势必会影响其他容器与外界的网络通信。对于有多个项目部署在`docker`上时，当一个项目需要频繁与外界网络通信时，就会影响到其他的项目网络，这显然不是我们想要的。举个例子，此时我们有三个项目，第一个项目`ems`有三个容器`mysql`、`redis`、`tomac`，第二个项目`dangdang`有`tomac`、`redis`、`mysql`三个容器，那当第一个项目频繁与外界通信，就会占用大量的网络资源，如此就会影响到第二个项目于外界的网络通信。因此这样的网桥机制对于我们来说是不够完美的。

因此`docker`设计了如下的网桥模式来帮助我们解决这个问题：

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210906152635999.png)

## docker网络使用

### 查看docker网桥配置

```
docker network ls
```

### 查看网桥详细信息

```
docker inspect [网桥名称]	//incpect不止能查看容器详细信息，也能查看网桥的，或许还有我不知道的，其功能通用而强大
```

### 创建自定义网桥

```
docker create [网桥名称]	==》		docker create -d（指定后台模式） bridge ems	==》后台创建一个名为ems的网桥
docker run -d -p 8081:8080 --network ems --name mytomac tomcat：8.0-jre8	==》启动一个容器，连接ems网桥
```

### 删除网桥

```
docker network rm [网桥名称]
```

### 注意：

> 一旦启动容器时制定了网桥之后，日后可以在任何这个网桥关联的容器中使用容器名字与其他容器通信

> 使用`docker run`指定`--network`网桥时网桥必须存在

# docker高级数据卷配置

## 数据卷详细

### 数据卷作用

前文有讲到数据卷的作用，就是为了实现容器与宿主机之间资源共享。

### 数据卷特点

1、数据卷可以在容器之间共享和重用

2、对数据卷的修改会立即影响到对应容器

3、对数据卷的更新修改，不会影响到镜像

4、数据卷会默认一直存在即使容器被删除

### 数据卷操作

1、自定义数据卷目录

```
docker -v 绝对路径：容器内路径
```

2、自动创建数据卷

```
docker run -v 卷名（随便起的自动创建）：容器内路径
```

### docker操作数据卷指令

1、查看数据卷

```
docker volume ls
```

2、查看数据卷细节

```
docker volume inspect 卷名
```

3、创建数据卷

```
docker volume create 卷名
```

4、删除没有使用的数据卷

```
docker volume prune //全部删除
docker volume rm 卷名 //指定删除
```

# dockerfile

## 什么是dockerfile

`dockerfile`可以认为是`docker`镜像的描述文件，是由一系列命令和参数构成的脚本。主要作用就是用来构建`docker`镜像的构建文件。

通过其架构图可以看出`dockerfile`可以直接构建镜像。

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210907104401541.png)

## 为什么要存在dockerfile

**问题:**在`dockerfile`中官方提供很多镜像已经基本能满足我们的所有需求了，为什么还需要自定义镜像呢？

**核心作用:**日后用户可以将自己的应用打包成镜像，这样就可以让我们的应用在容器中运行。

## dockerfile构建镜像的原理

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20210906193940570.png)

**构建过程**

`dockerfile`在接触到`build`指令后，会将`dockerfile`所在目录设置为上下文目录，一次性打包发送给`sever`端（如果有不想要发送给`sever`的文件或者目录，只需要将其改名为`.docker ignore`即可）。`server`端在接收到包之后，开始进行镜像构建。在构建的过程中，`dockerfile`的**每一行**都会生成一个临时镜像，临时生成的镜像会缓存在`cache`里面，如果`dockerfile`没有错误，那么`dockerfile`的最后一行生成的镜像就是我们最终构建的镜像。

**cache缓存的意义**

`cache`缓存的意义就是当我们的`dockerfile`编写有错误时，假设代码有30行，错误出现在第28行，那么当我们将28行的错误修改之后重新生成，系统会检测之前生成临时镜像的代码有没有修改，若没有修改，就会直接沿用之前的临时镜像，不会再生成临时镜像，只会在代码有修改的地方重新生成临时对象并缓存。因此`cache`缓存的存在是可以提高我们的生成镜像的效率的。当然，如果你想全部重新生成而不需要沿用之前缓存的临时镜像也是可以的，在`build`指令后面指定`--no-cache`即可。

## dockerfile的使用

| 命令       | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| FROM       | 当前镜像是基于的哪个镜像（第一条指令必须是FROM）             |
| RUN        | 构建镜像时需要运行的指令                                     |
| EXPOSE     | 当前容器对外暴露出的端口号                                   |
| WORKDIR    | 指定在创建容器后，终端默认登录进来的工作目录，一个落脚点     |
| ENV        | 用来在构建镜像过程中设置环境变量                             |
| ADD        | 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包 |
| COPY       | 类似于ADD，拷贝文件和目录到镜像中，将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置 |
| VOLUME     | 容器数据卷，用于数据保存和持久化工作                         |
| CMD        | 指定一个容器启动时要运行的命令。dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被docker run之后的参数替换 |
| ENTRYPOINT | 指定一个容器启动时要运行的命令。其目的是和CMD一样，都是在指定容器启动程序及其参数 |

### FROM命令

基于哪个镜像进行构建新的镜像，在构建时会自动从`docker hub`拉取`base`镜像。其必须作为`Dockerfile`的第一个指令出现。

语法：

```dockerfile
FROM <image>
FROM <image>[:<tag>]	使用版本不写为latest
FROM <image>[@<digest>]  使用摘要
```

### RUN命令

`RUN`命令将在当前镜像之上的新层中执行任何命令并提交结果。生成的提交镜像将用于`Dockerfile`中的下一步。

语法：

```dockerfile
RUN <command> (shell from, the command is run in a shell, which by default is /bin/bash -c on linux or cmd /S /C on Windows)
RUN echo hello

RUN ["executable","param1","param2"] (json数组形式)
RUN	["/bin/bash","-c","echo hello"]  (shell形式)
```

### EXPOSE命令

用来指定构造的镜像在运行为容器时对外暴露的端口

语法：

```dockerfile
EXPOSE 80/TCP    //若写的是EXPOSE 80，后面没接/XXX则默认为TCP
EXPOSE 80/UDP
```

### ENV指令

用来为构建镜像设置环境变量。这个值将会出现在构建阶段中所有后续指令的环境中。

语法：

```dockerfile
ENV <key> <value>
ENV	<key>=<value> ...
```

### COPY指令

用来将context目录中指定文件复制到镜像的制定目录中

语法：

```dockerfile
COPY src dest
COPY ["<src>",... "<dest>"]
```

### ADD命令

用来将context目录中指定文件复制到镜像的指定目录中

语法：

```dockerfile
ADD hom* /mydir/	通配符添加多个文件
ADD hom?.txt /mydir/ 通配符添加
ADD test.txt relativeDir/   可以指定相对路径
ADD test.txt /absoluteDir/   也可以指定绝对路径
ADD url
```

### WORKDIR命令

用来为`Dockerfile`中的任何`RUN、CMD、ENTERPOINT、COPY和ADD`指令设置工作目录。如果`WORKDIR`不存在，即使它没有在任何后续`Dockerfile`指令中使用，它也将被创建。当用户进入容器内时，会自动跳转到`WORKDIR`目录。

语法：

```dockerfile
//ex1：
WORKDIR /path/to/workdir
//ex2：
WORKDIR	/a
WORKDIR	b
WORKDIR	c
//WORKDIR指令可以在Dockerfile中多次使用，如果提供了相对路径，则该路径与先前WORKDIR指令的路径相对。例如上述则代表/a/b/c路径。
```

### VOLUME命令

用来定义容器运行时可以挂载到宿主机的目录，该值可以是json数组。

语法：

```dockerfile
VOLUME ["/data"]
```

### CMD命令

用来为启动的容器制动执行的命令，在`Dockerfile`中只能有一条CMD指令，如果有多条那就只有最后一条生效。

CMD可以被run后面的指令覆盖。

> ex：docker run mycentos7:1 echo "hello hfg"
>
> 在将镜像运行成容器时，在run指令后面加上其他命令即可覆盖dockerfile中的CMD指令。

语法：

```dockerfile
CMD ["executable","param1","param2"] (exec from, this is the preferred from)
CMD	["param1","param2"] (as default parameters to ENTERPOINT)
CMD command param1 param2 (shell from)
```

### ENTRYPOINT命令

用来指定容器启动时执行命令，和`CMD`类似。其也可以在镜像`run`生成容器的时候被指定命令覆盖。

> ex:docker run --entrypoint=ls mycentos7:2 /XXX/XXX
>
> 注意：entrypoint在覆盖时要加关键字指令才可以覆盖，且指令名称和指令本身是分开的

`ENTRYPOINT`指令往往用于设置容器启动后的**第一个命令**，这对于一个容器来说往往是固定的。

`CMD`指令，往往用于设置容器启动的第一个命令的**默认参数**，这对一个容器来说是可以变化的。

由于CMD指令覆盖相较`entrypoint`便捷，因此通常用`ENTRYPOINT`来作为**指令名称**，`CMD`用作**指令本身**。

```dockerfile
WORKDIR /usr/work
ENTRYPOINT ["tar","-xvf"]
CMD ["hello.tar.gz"]
```

> 假设dockerfile中有上述代码，作用是在工作目录/usr/work中解压压缩包。因此在终端时，我们运行镜像生成容器时会默认在工作目录解压hello的压缩包。那么问题来了，假设我们要解压的不是hello.tar.gz，而是hfg.tar.gz，怎么办呢？
>
> 由于CMD指令是可以在运行镜像生成容器时被覆盖的，因此我们只需要在运行镜像生成容器时添加参数覆盖dockerfile中的CMD指令即可。
>
> ex：docker run centos7:4 hfg.tar.gz

语法：

```dockerfile
ENTRYPOINT ["executable","param1","param2"]
ENTRYPOINT command param1 param2
```

# Docker compose

## 简介

`Compose`项目是`Docker`官方的开源项目，负责实现对`docker`容器或者集群的**快速编排**。从功能上看，跟`OpenStack`中的`Heat`十分类似。其代码目前在`https://github.com/docker/compose`上开源。

`Compose`定位是 `定义和运行多个Docker容器的应用` 。其前身是开源项目`Fig`。通过第一部分中的介绍，我们知道使用一个`Dockerfile`模板文件，可以让用户很方便的定义一个单独的应用容器。然而在日常工作中，经常会需要多个容器相互配合来完成某个项目。例如要实现一个`web`项目，除了`web`服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

`Compose`正好满足了这样的需求。它允许用户通过一个单独的`docker-compose.yml`模板文件（`YAML`格式）来定义一组相关联的应用容器为一个项目。

`Compose`中由两个重要的概念：

> 服务（service）：一个应用的容器，实际上可以包括若干个运行相同镜像的容器实例
>
> 项目（project）：由一组关联的应用容器组成的一个完整的业务单元，在docker-compose.yml文件中定义

`Compose`的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷的生命周期管理。

`Compose`项目由`python`编写，实现上调用了`docker`服务提供的`API`来对容器进行管理。因此，只要所操作的平台支持`Docker API`，就可以在其上利用`Compose`来进行编排管理。

## 安装与卸载

### linux

在`linux`上安装十分的简单，从官方`GitHub Release`处直接下载编译好的二进制文件即可。

在`Linux 64`位系统上直接下载对应的二进制包，将二进制包放入`/usr/local/bin`，并且改名为`docker-compose`

```bash
$ sudo curl -L https://github.com/docker/compose/releases/download/1.25.5/docker-compose-'uname -s'-'uname -m' > /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

### macos、windows

`docker-conpose`可以通过`python`的包管理工具`pip`进行安装，也可以直接下载编译好的二进制文件使用，甚至能够直接在`docker`容器中运行。`Docker Desktop for Mac/Windows` 自带`docker-compose`二进制文件，安装`docker`之后可以直接使用。

### bash命令补全

```bash
$ curl -L https://raw.githubusercontent.com/docker/compose/1.25.5/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```

### 卸载

如果是二进制包安装的直接删除二进制文件即可

```bash
$ sudo rm /usr/local/bin/docker-compose
```

测试安装成功

```bash
$ docker-compose --versiom    //查看docker版本，安装成功则显示当前docker版本
```

通过`docker-compose`运行一组容器

```bash
[root@centos ~]# docker-compose up    							//前台启动一组服务
[root@centos ~]# docker-compose up -d 							//后台启动一组服务
```

## docker-compose 模板文件

模板文件是使用 `Compose` 的核心，涉及到的指令关键字也比较多。这里面大部分指令跟 `docker run` 相关参数的含义都是类似的。

默认的模板文件名称为 `docker-compose.yml`，格式为 YAML 格式。

```yaml
version: "3"

services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```

注意每个服务都必须通过 `image` 指令指定镜像或 `build` 指令（需要 Dockerfile）等来自动构建生成镜像。

如果使用 `build` 指令，在 `Dockerfile` 中设置的选项(例如：`CMD`, `EXPOSE`, `VOLUME`, `ENV` 等) 将会自动被获取，无需在 `docker-compose.yml` 中重复设置。

下面分别介绍各个指令的用法。

### **`build`**

指定 `Dockerfile` 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 `Compose` 将会利用它自动构建这个镜像，然后使用这个镜像。

```yaml
version: '3'
services:

  webapp:
    build: ./dir
```

你也可以使用 `context` 指令指定 `Dockerfile` 所在文件夹的路径。

使用 `dockerfile` 指令指定 `Dockerfile` 文件名。

使用 `arg` 指令指定构建镜像时的变量。

```yaml
version: '3'
services:

  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

### **`command`**

覆盖容器启动后默认执行的命令。

```yaml
command: echo "hello world"
```

### **`container_name`**

指定容器名称。默认将会使用 `项目名称_服务名称_序号` 这样的格式。

```yaml
container_name: docker-web-container
```

> 注意: 指定容器名称后，该服务将无法进行扩展（scale），因为 Docker 不允许多个容器具有相同的名称。

### **`depends_on`**

解决容器的依赖、启动先后的问题。以下例子中会先启动 `redis` `db` 再启动 `web`

```yaml
version: '3'

services:
  web:
    build: .
    depends_on:
      - db
      - redis

  redis:
    image: redis

  db:
    image: postgres
```

> 注意：`web` 服务不会等待 `redis` `db` 「完全启动」之后才启动。

### **`env_file`**

从文件中获取环境变量，可以为单独的文件路径或列表。

如果通过 `docker-compose -f FILE` 方式来指定 Compose 模板文件，则 `env_file` 中变量的路径会基于模板文件路径。

如果有变量名称与 `environment` 指令冲突，则按照惯例，以后者为准。

```bash
env_file: .env

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

环境变量文件中每一行必须符合格式，支持 `#` 开头的注释行。

```bash
# common.env: Set development environment
PROG_ENV=development
```

### **`environment`**

设置环境变量。你可以使用数组或字典两种格式。

只给定名称的变量会自动获取运行 Compose 主机上对应变量的值，可以用来防止泄露不必要的数据。

```yaml
environment:
  RACK_ENV: development
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SESSION_SECRET
```

如果变量名称或者值中用到 `true|false，yes|no` 等表达 [布尔](https://yaml.org/type/bool.html) 含义的词汇，最好放到引号里，避免 YAML 自动解析某些内容为对应的布尔语义。这些特定词汇，包括

```bash
y|Y|yes|Yes|YES|n|N|no|No|NO|true|True|TRUE|false|False|FALSE|on|On|ON|off|Off|OFF
```

### **`healthcheck`**

通过命令检查容器是否健康运行。

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
```

### **`image`**

指定为镜像名称或镜像 ID。如果镜像在本地不存在，`Compose` 将会尝试拉取这个镜像。

```yaml
image: ubuntu
image: orchardup/postgresql
image: a4bc65fd
```

### **`networks`**

配置容器连接的网络。

```yaml
version: "3"
services:

  some-service:
    networks:
     - some-network
     - other-network

networks:
  some-network:
  other-network:
```

### **`ports`**

暴露端口信息。

使用宿主端口：容器端口 `(HOST:CONTAINER)` 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

```yaml
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```

**注意**：当使用 `HOST:CONTAINER` 格式来映射端口时，如果你使用的容器端口小于 60 并且没放到引号里，可能会得到错误结果，因为 `YAML` 会自动解析 `xx:yy` 这种数字格式为 60 进制。为避免出现这种问题，建议数字串都采用引号包括起来的字符串格式。

### **`sysctls`**

配置容器内核参数。

```yaml
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0

sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

### **`ulimits`**

指定容器的 `ulimits` 限制值。

例如，指定最大进程数为 65535，指定文件句柄数为 20000（软限制，应用可以随时修改，不能超过硬限制） 和 40000（系统硬限制，只能 `root` 用户提高）。

```yaml
  ulimits:
    nproc: 65535
    nofile:
      soft: 20000
      hard: 40000
```

### **`volumes`**

数据卷所挂载路径设置。可以设置为宿主机路径(`HOST:CONTAINER`)或者数据卷名称(`VOLUME:CONTAINER`)，并且可以设置访问模式 （`HOST:CONTAINER:ro`）。

该指令中路径支持相对路径。

```yaml
volumes:
 - /var/lib/mysql
 - cache/:/tmp/cache
 - ~/configs:/etc/configs/:ro
```

如果路径为数据卷名称，必须在文件中配置数据卷。

```yaml
version: "3"

services:
  my_src:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

---

## docker-compose 常用命令

### 命令对象与格式

对于 `Compose` 来说，大部分命令的对象既可以是项目本身，也可以指定为项目中的服务或者容器。如果没有特别的说明，命令对象将是项目，这意味着项目中所有的服务都会受到命令影响。

执行 `docker-compose [COMMAND] --help` 或者 `docker-compose help [COMMAND]` 可以查看具体某个命令的使用格式。

`docker-compose` 命令的基本的使用格式是

```bash
docker-compose [-f=<arg>...] [options] [COMMAND] [ARGS...]
```

### 命令选项

> - `-f, --file FILE` 指定使用的 Compose 模板文件，默认为 `docker-compose.yml`，可以多次指定。
> - `-p, --project-name NAME` 指定项目名称，默认将使用所在目录名称作为项目名。
> - `--x-networking` 使用 Docker 的可拔插网络后端特性
> - `--x-network-driver DRIVER` 指定网络后端的驱动，默认为 `bridge`
> - `--verbose` 输出更多调试信息。
> - `-v, --version` 打印版本并退出。
>

### 命令使用说明

#### **`up`**

格式为 `docker-compose up [options] [SERVICE...]`。

> - 该命令十分强大，它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。
>
> - 链接的服务都将会被自动启动，除非已经处于运行状态。
>
> - 可以说，大部分时候都可以直接通过该命令来启动一个项目。
>
> - 默认情况，`docker-compose up` 启动的容器都在前台，控制台将会同时打印所有容器的输出信息，可以很方便进行调试。
>
> - 当通过 `Ctrl-C` 停止命令时，所有容器将会停止。
>
> - 如果使用 `docker-compose up -d`，将会在后台启动并运行所有的容器。一般推荐生产环境下使用该选项。
>
> - 默认情况，如果服务容器已经存在，`docker-compose up` 将会尝试停止容器，然后重新创建（保持使用 `volumes-from` 挂载的卷），以保证新启动的服务匹配 `docker-compose.yml` 文件的最新内容
>

---

#### **`down`**

> - 此命令将会停止 `up` 命令所启动的容器，并移除网络
>

----

#### **`exec`**

> - 进入指定的容器。
>

----

**`ps`**

格式为 `docker-compose ps [options] [SERVICE...]`。

列出项目中目前的所有容器。

选项：

> - `-q` 只打印容器的 ID 信息。
>

----

#### **`restart`**

格式为 `docker-compose restart [options] [SERVICE...]`。

重启项目中的服务。

选项：

> - `-t, --timeout TIMEOUT` 指定重启前停止容器的超时（默认为 10 秒）。
>

----

#### **`rm`**

格式为 `docker-compose rm [options] [SERVICE...]`。

删除所有（停止状态的）服务容器。推荐先执行 `docker-compose stop` 命令来停止容器。

选项：

> - `-f, --force` 强制直接删除，包括非停止状态的容器。一般尽量不要使用该选项。
> - `-v` 删除容器所挂载的数据卷。
>

---

#### **`start`**

格式为 `docker-compose start [SERVICE...]`。

启动已经存在的服务容器。

----

#### **`stop`**

格式为 `docker-compose stop [options] [SERVICE...]`。

停止已经处于运行状态的容器，但不删除它。通过 `docker-compose start` 可以再次启动这些容器。

选项：

> - `-t, --timeout TIMEOUT` 停止容器时候的超时（默认为 10 秒）。
>

----

#### **`top`**

查看各个服务容器内运行的进程。

---

#### **`unpause`**

格式为 `docker-compose unpause [SERVICE...]`。

恢复处于暂停状态中的服务。

------



## docker-compose.yml示例文件

```yml
#版本，目前支持的最新版本为3.8，最好使用3.2到3.8版本。dockers引擎版本对应支持的compose版本可以在docker官网中查询。
version: "3.8"  
service: 	#“服务”的名称
    ubuntu:
    image: ubuntu:15.10			 #指定使用的镜像
    container_name: ubuntu		 #指定启动时容器的名称
    ports:						#指定容器端口与宿主机端口映射
     - 192.168.218.130:"8080:8080"		    	
    volumes:					#指定容器中哪个路径与宿主机中路径进行数据映射
     - /root/usr:/usr/local/work
     //- workspace:/usr/local/work
    networks:					#指定容器启动之后使用哪个网桥
     - aa
    cmd: redis-server /......	  #用来覆盖容器默认启动指令
    envoriment: 				#用来指定容器启动时环境参数		
     - MYSQL_ROOT_PASSWORD=root
     //- MYSQL_ROOT_PASSEORD root
volume:
  workspace:
    external:					#要使用外部的数据卷则需要设置external为true，不然是只读的
      true
      
network:
  aa:
    external:					#同理，需要用到外部网桥的话需要设置external为true
      true
```

# docker可视化工具

##  安装Portainer

官方安装说明：https://www.portainer.io/installation/

```
[root@ubuntu1804 ~]#docker pull  portainer/portainer

[root@ubuntu1804 ~]#docker volume create portainer_data
portainer_data
[root@ubuntu1804 ~]#docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
20db26b67b791648c2ef6aee444a5226a9c897ebcf0160050e722dbf4a4906e3
[root@ubuntu1804 ~]#docker ps 
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS                                            NAMES
20db26b67b79        portainer/portainer   "/portainer"        5 seconds ago       Up 4 seconds        0.0.0.0:8000->8000/tcp, 0.0.0.0:9000->9000/tcp   portainer
```

## 登录和使用Portainer

> 用浏览器访问：`http://localhost:9000`

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/image-20201223231707738.png)

------

# docker镜像下载到本地，并在其他机器恢复

## 查看镜像

```
docker images
```

## 保存到本地后拷贝到目标机器

```
docker save 24cc0d9dcf54 > /root/xxl.tar	  //24cc0d9dcf54为镜像ID
```

## 加载镜像到docker

```
docker load < /root/xxl.tar	 
```

## 查看目标机器镜像

> 加载成功后这两个地方会是none，需要我们修改标签

![img](https://raw.githubusercontent.com/gdcgx/picture/main/img/9101119-ed29d3fd87f007f1.png)

## 修改当前机器镜像标签

```
docker tag 24cc0d9dcf54 xxl-job:latest
```

然后就能使用`docker run`命令来启动了

