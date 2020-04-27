# DOCKER

<!-- TOC -->

- [DOCKER](#docker)
    - [容器和虚拟机](#容器和虚拟机)
    - [Docker的优势](#docker的优势)
        - [Docker相比于传统虚拟化方式具有更多的优势:](#docker相比于传统虚拟化方式具有更多的优势)
    - [docker的三个基本概念](#docker的三个基本概念)
        - [Image（镜像）](#image镜像)
        - [Container（容器）](#container容器)
        - [Repository (仓库)](#repository-仓库)
    - [Docker的安装和使用](#docker的安装和使用)
    - [Docker架构](#docker架构)
        - [Docker Client](#docker-client)
        - [Docker daemon](#docker-daemon)
        - [Docker Image](#docker-image)
        - [Docker Registry](#docker-registry)
        - [Docker Container](#docker-container)
    - [Docker组件是如何协运行容器](#docker组件是如何协运行容器)
    - [Docker常用命令](#docker常用命令)
    - [Dockerfile是什么](#dockerfile是什么)
    - [Dockerfile常用的指令](#dockerfile常用的指令)
        - [FROM](#from)
        - [MAINTAINER](#maintainer)
        - [COPY](#copy)
        - [WORKDIR](#workdir)
        - [RUN](#run)
        - [EXPOSE](#expose)
        - [ENTRYPOINT](#entrypoint)
        - [CMD](#cmd)

<!-- /TOC -->
## 容器和虚拟机

我们用的传统虚拟机如VMware，VisualBox之类的需要模拟整台机器包括硬件，每台虚拟机都需要有自己的操作系统，虚拟机一旦被开启，预分配给它的资源将全部被占用。每一台虚拟机包括应用，必要的二进制和库，以及一个完整的用户操作系统。  

而容器技术是和我们的宿主机共享硬件资源及操作系统，可以实现资源的动态分配。容器包含应用和其所有的依赖包，但是与其他容器共享内核。容器在宿主机操作系统中，在用户空间以分离的进程运行。  

容器技术是实现操作系统虚拟化的一种途径，可以让在资源受到隔离的进程中运行应用程序及其依赖关系。通过使用容器，我们可以轻松打包应用程序的代码、配置和依赖关系，将其变成容易使用的构建块，从而实现环境一致性、运营效率、开发人员生产力和版本控制等诸多目标。容器可以保住保证应用程序快速、可靠、一致的部署，其间不受部署环境的影响。容器还赋予我们对资源更多的精细化控制能力，让我们的基础设施效率更高。通过下面这幅图我们可以很直观的反映出这两者的区别所在

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181010205426157-1788702025.png)

（Hypervisor，又称虚拟机监视器（英语：virtual machine monitor，缩写为 VMM），是用来建立与执行虚拟机器的软件、固件或硬件。）

**Docker属于Linux容器的一种封装，提供简单易用的容器使用接口。** 它是目前最流行的Linux容器解决方案。  

而Linux容器是Linux发展出的另一种虚拟化技术，简单的讲，linux容器不是模拟一个完整的操作系统，而是对进程进行隔离，相当于是在正常进程的外边套了一个保护层。对于容器里边的进程来说，他接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。  

Docker将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器中运行，就好像在真实的物理机上运行一样。有了Docker，就不用担心环境问题。

总体来说,Docker的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通代码一样

## Docker的优势

### Docker相比于传统虚拟化方式具有更多的优势:

- docker启动快速属于秒级别。虚拟机通常需要几分钟去启动
- docker需要的资源更小,docker在操作系统级别进行虚拟化，docker容器和内核交互，几乎没有性能损耗，性能优于通过Hypervisor层与内核层的虚拟化
- docker更轻量，docker的架构可以共用一个内核与共享应用程序库，所占内存极小。同样的硬件环境，Docker运行的镜像源多于虚拟机数量，对系统的利用率非常高
- 与虚拟机相比，docker隔离性更弱，docker属于进程之间的隔离，虚拟机可实现系统级别隔离
- 安全性：docker的安全性也更弱。Docker的租户root与宿主机的root等同，一旦容器内的操作从普通用户提升为root权限，它就直接具备了宿主机的root权限，进而可以进行无限制的操作。虚拟机租户 root 权限和宿主机的 root 虚拟机权限是分离的，并且虚拟机利用如 Intel 的 VT-d 和 VT-x 的 ring-1 硬件隔离技术，这种隔离技术可以防止虚拟机突破和彼此交互，而容器至今还没有任何形式的硬件隔离，这使得容器容易受到攻击
- 可管理性：docker的集中化管理工具还不算成熟。各种虚拟化技术都有成熟的管理工具，例如VMware vCenter 提供完备的虚拟机管理能力
- 高可用和可恢复性：docker对业务的高可用支持是通过快速部署实现的。虚拟化具备负载均衡，高可用，容错，迁移和数据保护等经过生产实践检验的成熟保障机制，VMware可承诺虚拟机99.999%高可用，保证业务连续性
- 快速创建、删除：虚拟化创建是分钟级别的，Docker容器创建是秒级别的，Docker的快速迭代性，决定了不论是开发、测试、部署都可以节约大量时间
- 交付、部署：虚拟机可以通过镜像实现环境交付的一致性，但镜像分发无法体系化。docker在DockerFile中记录了容器构建过程，可在集群中实现快速分发和快速部署

我们可以从下面的这张表格很清楚的看到容器相比于传统虚拟机的特性的优势所在：
|  特性   | 容器  |虚拟机  |
|  :----  | :----  |:----  |
| 启动  | 秒级 |分钟级 |
| 硬盘使用  | 一般为MB |一般为GB |
| 性能  | 接近原生 |弱于 |
| 系统支持量  | 单机支持上千个容器 |一般是几十个 |

## docker的三个基本概念

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181010205425908-509725301.jpg)

从上图我们可以看到，docker中包括三个基本的概念：

- Image（镜像）
- Container（容器）
- Repository（仓库）

镜像是docker运行容器的前提，仓库是存放镜像的仓所，可见镜像是格式Docker的核心

### Image（镜像）

Docker镜像可以看做是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像包含任何动态数据，其内容在构建之后也不会改变。

镜像（Image）就是一堆只读层（read-only layer）的统一视角，也许这个定义游戏难以理解，西面这张图能够帮助理解镜像的定义。

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181010205425698-1711765011.png)

从左边我们看到了多个只读层，他们重叠在一起。除了最下面一层，其它层都有一个指针指向下一层。这些曾是Docker内部的实现细节，并且能够在主机的文件系统上访问到。统一文件系统(union file system)技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。我们可以在图片的右边看到这个视角的形式。

### Container（容器）

容器（container）的定义和镜像（image）几乎一模一样，也是一堆层的统一视角，唯一区别在于最上面那一层是可读可写的。
![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181010205425262-960721404.png)

由于容器的定义并没有提及是否要运行容器，所以实际上，容器=镜像+读写层。

### Repository (仓库)

Docker仓库是集中存放镜像文件的场所。镜像构建完成后，可以很容易的在当前宿主上运行，但是，如果需要在其他服务器上使用这个镜像，我们就需要一个集中地存储、分发镜像的服务，Docker Registry（仓库注册服务器）就是这样的服务。有时候会把仓库(Repository)和仓库注册(Registry)混为一谈，并不严格区分。Docker仓库的概念跟git相似，注册服务器可以理解为Github这样的托管服务。实际上，一个Docker Registry中可以包含多个仓库（Repository），每个仓库可以包含多个标签（tag），每个标签对应着一个镜像。所以说，镜像仓库是docker用来集中存放镜像文件的地方类似于我们之前常用的代码仓库。

通常，**一个仓库会包含同一个软件不同版本的镜像**，而**标签就常用于对应该软件的各个版本**。我们可以通过<仓库名>:<标签>的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以latest作为默认标签。

仓库又可以分为两种形式：

- public（共有仓库）
- private（私有仓库）

Docker Registry共有仓库是开放给用户使用、允许用户管理镜像的Registry服务，一般这类公开服务允许用户免费上传、下载公开的镜像，并可能提供收费服务供用户管理的私有镜像。

除了使用公开服务外，用户还可以在本地搭建私有Docker Registry。 Docker官方提供了Docker Registry镜像，可以直接使用作为私有Registry服务。当用户创建了自己的镜像之后就可以使用push命令将它上传到共有或者私有仓库，这样下次在另外一台服务器上使用这个镜像的时候，只需要从仓库上pull下来就可以了。

这里主要把docker的一些常见概念如Image，Container，Repository做了详细的阐述，也从传统虚拟化的角度阐述了docker的优势，我们从下图可以直观的看到Docker的架构：

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181011200344086-1510826338.png)

Docker使用C/S结构，即**客户端/服务器**体系结构。Docker客户端与Docker服务器进行交互，Docker服务端负责创建、运行和分发Docker镜像，Docker客户端和服务端可以运行在一台机器上，也可以通过RESTful、stock或网络接口与远程Docker服务端进行通信。

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181011200343656-1972949758.png)

这张图展示了Docker客户端、服务端和Docker仓库（即Docker Hub和Docker Cloud），默认情况下Docker会在Docker中央仓库寻找镜像文件，这种里利用仓库管理镜像的设计理念类似于Git，当然这个仓库是可以通过修改配置来指定的，甚至我们可以创建我们自己的私有仓库。

## Docker的安装和使用

Docker的安装和使用有一些前提条件，主要体现在体系架构和内核的支持上。对于体系架构，出了Docker一开始就支持的X86-64，其他体系架构的支持则一直在不断地完善和推进中。

Docker分为CE和EE两大版本。CE即社区版（免费，支持周期7个月），EE即企业版，强调安全，付费使用，支持周期24个月。

我们在安装前可以参看官方文档获取最新的Docker支持情况，官方文档在这里：

    https://docs.docker.com/install/

Docker对于内核支持的功能，即内核的配置选项也有一定的要求（比如必须开启Cgroup和Namespace相关选项，以及其他的网络和存储驱动等），Docker源码中提供了一个检测脚本和指导内核的配置，脚本连接在这里：

    https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh

在满足前提条件后，安装就变得非常简单了。

Docker CE的安装请参考官方文档：

- MacOS：<https://docs.docker.com/docker-for-mac/install/>
- Windows：<https://docs.docker.com/docker-for-windows/install/>
- Ubuntu：<https://docs.docker.com/install/linux/docker-ce/ubuntu/>
- Debian：<https://docs.docker.com/install/linux/docker-ce/debian/>
- CentOS：<https://docs.docker.com/install/linux/docker-ce/centos/>
- Fedora：<https://docs.docker.com/install/linux/docker-ce/fedora/>
- 其他Linux发行版：<https://docs.docker.com/install/linux/docker-ce/binaries/>

安装docker：

在测试环境或开发环境中Docker官方为了简化安装流程，提供了一套便捷的安装脚本，Ubuntu系统上可以使用这套脚本安装：

```sh bash
curl -fsSL get.docker.com -o get-docker.sh
bash get-docker.sh
```

具体可以参看docker-install的脚本：

    https://github.com/docker/docker-install

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把Docker CE的Edge版本安装在系统中。

安装完成后，运行下面的命令，验证是否安装成功：

```bash
    docker version
    or
    docker info
```
返回docker的版本相关信息，证明docker安装成功

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181011200342486-1791743202.png)

启动Docker-CE

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Docker的简单运用--Hello World

我们直接运行下面的命令，将名为hello-world的image文件从仓库抓取到本地。

```bash
docker pull library/hello-world
```

docker pull image 是拉取image文件，library/hello-world是image文件在仓库里边的位置，library是image文件所在的组，hello0world是image文件的名字。

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181011200342100-457982713.png)

拉取成功后，就可以在本机看到这个image文件了。

```bash
docker images
```

我们可以看到如下结果：

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181011200341767-414070160.png)

现在，我们可以运行hello-world这个image文件

```bash
docker run hello-world
```

我们可以看到如下结果：

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181011200341270-1701862326.png)

输出这段提示以后，hello world就会停止运行，容器自动终止。有些容器不会自动终止，因为提供的是服务，比如MySQL镜像等

Docker提供了一套简单实用的命令来创建和更新镜像，我们可以通过网络直接下载一个已经创建好了的应用镜像，并通过Docker RUN命令就可以直接使用。当镜像通过RUN命令运行成功后，这个运行的镜像就是一个Docker容器，容器可以理解为一个轻量级的沙箱，Docker利用容器来运行和隔离应用，容器是可以被启动、停止、删除的，这并不会影响Docker镜像。

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181011200340694-1202370283.png)

Docker客户端是Docker用户与Docker交互的主要方式，当使用Docker命令行运行命令时，Docker客户端将这些命令发送给服务端，服务端将执行这些命令，Docker命令使用Docker API。Docker客户端可以与多个服务端进行通信。

## Docker架构

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181012214103608-969706945.png)

Docker的核心组件包括：

1. Docker Client
2. Docker daemon
3. Docker Image
4. Docker Registry
5. Docker Container

Docker采用的是Client/Server架构。客户端向服务端发送请求，服务器负责构建、运行和分发容器。客户端和服务器可以运行在同一Host上，客户端也可以通过Socket或REST API 与远程的服务器进行通信。

### Docker Client

Docker Client，也称Docker客户端。他其实就是Docker提供命令行界面（CLI）工具，是许多Docker用户与Docker进行交互的主要方式。客户端可以构建、运行和停止应用程序，还可以远程与Docker_Host进行交互。最常用的Docker客户端就是Docker命令，我们可以通过Docker命令很方便的在host上构建和运行docker容器。

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181012214102869-1003920118.png)

### Docker daemon

Docker daemon是服务器组件，以Linux后台服务的方式运行，是Docker最核心的后台进程，我们也把它成为守护进程。它负责相应来自Docker Client的请求，然后将这些请求翻译成系统调用完成容器管理工作。该进程会在后台启动一个API Server，负责接收由Docker Client发送的请求，接收到的请求将通过Docker daemon内部的一个路由分发调度，由集体的函数来执行请求。

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181012214101750-1118997362.png)

我们大致可以将其分为以下三部分：

- Docker Server
- Engine 
- Job

Docker Daemon的架构如下所示：

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181012214101184-1339527466.jpg)

Docker Daemon可以认为是通过Docker Server模块接受Docker Client的请求，并在Engine中处理请求，然后根据请求类型，创建出指定的Job并运行。Docker Daemon运行在Docker host上，负责创建、运行、监控容器、构建、存储镜像。

运行过程的作用有以下几种可能：

- 向Docker Registry获取镜像
- 通过 graphdriver 执行容器镜像的本地化操作
- 通过 networkdriver 执行容器网络环境的配置
- 通过 execdriver 执行容器内部运行的执行工作

由于Docker Daemon和Docker Client的启动都是通过可执行文件docker来完成的，因此两者的启动流程非常相似。Docker可执行文件运行时，运行代码通过不同的命令行flag参数，区分两者，并最终运行两者各自相应的部分。

启动Docker Daemon时，一般可以使用以下命令来完成

```bash
docker --daemon = true
docker –d
docker –d = true
```

再由Docker的main（）函数来解析以上命令的对应flag参数，并最终完成Docker daemon的启动。

下图可以很直观的看到Docker Daemon的启动流程：

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181012214100787-426925315.jpg)

默认配置下，Docker daemon只能响应来自本地host的客户端请求。如果允许远程客户端请求，需要在配置文件中打开TCP监听。我们可以照着如下步骤进行配置：

1. 编辑配置文件/etc/systemd/system/multi-user.target.wants/docker.service ，在环境变量ExecStart后面添加 -H tcp://0.0.0.0，允许来自任意 IP 的客户端连接。

    ![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181012214100440-905260954.jpg)

2. 重启Docker Daemon
    
    ```bash
    systemctl daemon-reload
    systemctl restart docker.service
    ```
3. 我们通过以下命令即可实现与远程服务器通信

    ```bash
    docker -H 服务器IP地址 info
    ```

### Docker Image

docker镜像可以看做是一个特殊的文件系统，除了提供容器运行是所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会改变。我们可将Docker镜像看成只读模板，通过它可以创建Docker容器。

镜像有多种生成方法：

1. 从无到有开始创建镜像
2. 下载并使用别人创建好的现成的镜像
3. 在现有镜像上创建新的镜像

我们可以将镜像的内容和创建步骤描述在一个文件中，这个文件被称作Dockerfile，通过执行docker build &lt;docker-file&gt; 命令可以构建出Docker镜像。

### Docker Registry

Docker Registry 是存储docker image的仓库，它在Docker生态环境中的位置如下图所示：

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181012214100277-775657574.jpg)

运行docker push、docker pull、docker search时，实际上是通过 docker daemon 与 docker registry 通信。

### Docker Container

Docker容器就是Docker镜像的运行实例，是真正运行项目程序、消耗系统资源、提供服务的地方。Docker Container提供了系统硬件环境，我们可以使用Docker Images这些制作好的系统盘，再加上我们所编写好的项目代码，run一下就可以运行服务了

## Docker组件是如何协运行容器

容器启动过程如下：

- Docker客户端执行docker run命令
- Docker Daemon发现本地没有hello-world镜像
- daemon从Docker hub下载镜像
- 下载完成，镜像hello-world被保存到本地
- Docker daemon 启动容器

具体过程可以看如下这幅演示图

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181012214059835-734054348.png)

我们可以通过docker images可以查看到hello-world 已经下载到本地

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181012214059170-1387452245.png)

我们可以通过docker ps或者docker container ls现在正在运行的容器，我们可以看到，hello-world在输出显示提示信息之后就会停止运行，容器自动停止，所以我们在查看的时候没有发现有容器在运行。

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181012214058803-1594502697.png)

## Docker常用命令

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181014202945937-1677031749.png)

例如，我们需要拉取一个docker镜像，我们可以用如下命令：

```bash
docker pull image_name
```

image_name 为镜像的名称，而如果我们想从Docker hub上去下载某个镜像，我们可以使用以下命令：

```bash
docker pull ubuntu:latest
```

ubuntu:lastest 是镜像的名称， Docker daemon 发现本地没有我们需要的镜像，会自动去 Docker Hub 上去下载镜像，下载完成后，该镜像被默认保存到 /var/lib/docker 目录下。

接着我们如果想查看下主机下存在多少镜像，我们可以用如下命令：

```bash
docker images
```

我们要想知道当前有哪些容器在运行，我们可以用如下命令：

```bash
docker ps -a
```

-a 是查看当前所有的容器，包括未运行的

对一个容器进行启动，重启和停止，可以使用如下命令：

```bash
docker start container_name/container_id
docker restart container_name/container_id
docker stop container_name/container_id
```

这个时候我们如果想进入到这个容器中，我们可以使用attach命令：

```bash
docker attach container_name/container_id
```

如果想运行这个容器中的镜像的话，并且调用镜像里边的bash，我们可以使用如下命令：

```bash
docker run -t -i container_name/container_id /bin/bash
```

如果想删除指定镜像的话，由于image被某个container引用（拿来运行），如果不将这个引用的container销毁（删除），那image肯定不能被删除。我们首先得先去停止这个容器：

```bash
docker ps
docker stop container_name/container_id
```

然后我们用如下命令去删除这个容器：

```bash
docker rm container_name/container_id
```

然后这个时候我们再去删除这个镜像：

```bash
docker rmi image_name
```

## Dockerfile是什么

Dockerfile是自动构建docker镜像的配置文件，用户可以使用Dockerfile快速创建自定义镜像。Dockerfile中的命令非常类似与Linux下的shell命令

我们可以通过下面这幅图来直观的感受下Docker镜像、容器、和Dockerfile三者之间的关系。

![avatar](https://img2018.cnblogs.com/blog/1100338/201810/1100338-20181014202945505-1291953865.png)

从上图我们可以看到，Dockerfile可以自定义镜像，通过Docker命令去运行镜像，从而达到启动容器的目的。

Dockerfile是由一行行命令语句组成，并且支持以#开头的注释行。

一般来说，我们可以将Dockerfile分为四个部分：

- **基础镜像（父镜像）指令FROM**
- **维护者信息指令MAINTAINER**
- **镜像操作指令 RUN 、 EVN 、 ADD 和 WORKDIR 等**
- **容器启动指令 CMD 、 ENTRYPOINT 和 USER 等**

下面是一段简单的Dockerfile的例子：

```bash
FROM python:2.7
MAINTAINER Snax <snax7355608@gmail.com>
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 5000
ENTRYPOINT ["python"]
CMD ["app.py"]
```

我们可以分析一下上面这个过程

1. 从DockerHub上pull下Python 2.7的基础镜像
2. 显示维护者信息
3. copy当前目录到容器中的/app目录下 复制本地主机&lt;src&gt;(Dockerfile所在目录的相对路径)到容器&lt;dest&gt;
4. 指定工作路径为/app
5. 安装依赖包
6. 暴露5000端口
7. 启动app

这例子是启动一个Python flask app的DockerFile（flask是python的一个轻量的web框架）

## Dockerfile常用的指令

由于Dockerfile中所有的命令都是以下格式：INSTRUCTION argument ，指令（INSTRUCTION）不区分大小写，但是推荐大写，和sql语句类似。

### FROM

FORM是用于指定基础的images，一般格式为 FROM &lt;image&gt; or FORM &lt;image&gt;:&lt;tag&gt; ，所有的 Dockerfile 都用该以 FROM 开头，FROM 命令指明 Dockerfile 所创建的镜像文件以什么镜像为基础，FROM 以后的所有指令都会在 FROM 的基础上进行创建镜像。可以在同一个 Dockerfile 中多次使用 FROM 命令用于创建多个镜像。比如我们要指定 python 2.7 的基础镜像，我们可以像如下写法一样：

```bash
FROM python:2.7
```

### MAINTAINER

MAINTAINER是用于指定镜像创建者和联系方式，一般格式为 MAINTAINER &lt;name&gt; 。

```bash
MAINTAINER snax <snax7355608@gmail.com>
```

### COPY

COPY 是用于复制本地主机的 &lt;src&gt; (为 Dockerfile 所在目录的相对路径)到容器中的 &lt;dest&gt;。

当使用本地目录为源目录时，推荐使用 COPY 。一般格式为 COPY &lt;src&gt;&lt;dest&gt; 。例如我们要拷贝当前目录到容器中的 /app 目录下，我们可以这样操作：

```bash
COPY . /app
```

### WORKDIR

WORKDIR 用于配合 RUN，CMD，ENTRYPOINT 命令设置当前工作路径。可以设置多次，如果是相对路径，则相对前一个 WORKDIR 命令。默认路径为/。一般格式为 WORKDIR /path/to/work/dir 。例如我们设置/app 路径，我们可以进行如下操作:

```bash
WORKDIR /app
```

### RUN

RUN用于容器内部执行命令。每个RUN命令相当于在原有的镜像基础上添加了一个改动层，原有的镜像不会有变化。一般格式为 RUN &lt;command&gt; 。例如我们要安装 python 依赖包，我们做法如下：

```bash
RUN pip install -r requirements.txt
```

### EXPOSE

EXPOSE 命令用来指定对外开放的端口。一般格式为 EXPOSE &lt;port&gt; [&lt;port&gt;...]
例如上面那个例子，开放5000端口：

```bash
EXPOSE 5000
```

### ENTRYPOINT

ENTRYPOINT 可以让你的容器表现得像一个可执行程序一样。一个 Dockerfile 中只能有一个 ENTRYPOINT，如果有多个，则最后一个生效。

ENTRYPOINT 命令也有两种格式：

- ENTRYPOINT ["executable", "param1", "param2"] ：推荐使用的 exec形式
- ENTRYPOINT command param1 param2 ：shell 形式

例如若要将python镜像变成可执行的程序：

```bash
ENTRYPOINT ["python"]
```

### CMD

CMD 命令用于启动容器时默认执行的命令，CMD 命令可以包含可执行文件，也可以不包含可执行文件。不包含可执行文件的情况下就要用 ENTRYPOINT 指定一个，然后 CMD 命令的参数就会作为ENTRYPOINT的参数。

CMD 命令有三种格式：

- CMD ["executable","param1","param2"]：推荐使用的 exec 形式。
- CMD ["param1","param2"]：无可执行程序形式
- CMD command param1 param2：shell 形式。

一个Dockerfile中只能有一个CMD，如果有多个，则最后一个生效。而CMD的shell形式默认调用/bin/sh -c 执行命令。

CMD命令会被Docker命令行传入的参数覆盖： docker run busybox /bin/echo Hello Docker 会把 CMD 里的命令覆盖。

例如我们要启动 /app， 我们可以用如下命令实现：

```bash
CMD ["app.py"]
```
