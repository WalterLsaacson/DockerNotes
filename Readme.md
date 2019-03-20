#### docker学习笔记

* docker 不同于其他虚拟化技术的关键点

  Docker 使用 Google 公司推出的 [Go 语言](https://golang.org/) 进行开发实现，基于 Linux 内核的 [cgroup](https://zh.wikipedia.org/wiki/Cgroups)，[namespace](https://en.wikipedia.org/wiki/Linux_namespaces)，以及[AUFS](https://en.wikipedia.org/wiki/Aufs) 类的 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 等技术，对进程进行封装隔离，属于 [操作系统层面的虚拟化技术](https://en.wikipedia.org/wiki/Operating-system-level_virtualization)。

  传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

* ### 基本概念

  * #### 镜像(Image)

    * Linux系统在内核启动之后会挂载root文件系统提供用户空间支持。而docker镜像就相当于是一个最小系统的root文件系统
    * Docker镜像提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）
    * **分层存储**
    * docker在设计是充分利用Union FS技术，将其设计为分层存储的架构
    * 分层架构在实际使用中的体现？

  * #### 容器(Container)

    * 镜像和容器的关系，就像是面向对象程序设计中的类和实例一样，竟像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
    * 容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，称这个为容器运行时读写准备的存储层为容器存储层。
    * 容器在运行时所进行的文件写入操作都应该是针对数据卷、或者绑定宿主目录，将是直接对宿主计算机的文件读写操作，其性能和稳定性会更高。

  * #### 仓库(Repository)

    * docker registry 中可以包含多个仓库，每个仓库可以包含多个标签，每个标签对应一个镜像。例如：ubuntu:18.04为一个《仓库名：标签》的典型形式，如果不指定标签ubuntu这样的形式将被识别为ubuntu:latest
    * registry公开服务：官方的[Docker Hub](https://hub.docker.com/)，还有 CoreOS,Google,Kubernetes等镜像。
    * 私有registry

* ### 安装docker(Ubuntu系统)

  Docker 分为 CE 和 EE 两大版本。CE 即社区版（免费，支持周期 7 个月），EE 即企业版，强调安全，付费使用，支持周期 24 个月。

  Docker CE 分为 **stable**, **test**, 和 **nightly** 三个更新频道。每六个月发布一个 **stable** 版本 (18.09, 19.03, 19.09...)。

  * Linux版本 LTS（long-term-support）版本
  * 卸载旧版本的docker

  ```shell
  $ sudo apt-get remove docker \
                 docker-engine \
                 docker.io
  ```

  * Ubuntu 14.04可选内核模块

  ```shell
  sudo apt-get update
  sudo apt-get install \
  	linux-image-extra-$(uname -r) \
      linux-image-extra-virtual
  ```

  * Ubuntu 16.04 以上的Docker CE 默认使用overlay2 存储层驱动，无需手动配置

  * 使用APT安装

    由于apt源使用HTTPS以确保软件下载过程中不被篡改。因此，我们首先需要添加使用HTTPS传输的软件包以及CA证书。

    ```shell
    $ sudo apt-get update
    
    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
    ```

    鉴于国内网络问题，强烈建议使用国内源.

    为了确认所下载软件包的合法性，需要添加软件源的GPG密钥。

    ```shell
    $ curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
    
    
    # 官方源
    # $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

    然后我们需要向source.list中添加Docker软件源

    ```shell
    $ sudo add-apt-repository \
        "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
        $(lsb_release -cs) \
        stable"
    
    
    # 官方源
    # $ sudo add-apt-repository \
    #    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    #    $(lsb_release -cs) \
    #    stable"
    ```

    > 以上命令会添加稳定版本的Docker CE APT镜像源，如果需要测试或者每日构建版本的Docker CE请将stable改为test或者nightly

    #### 安装Docker CE

    ```shell
    sudo apt-get update
    sudo apt-get install docker-ce
    ```

    #### 使用脚本自动安装

    在测试或者开发环境中Docker官方为了简化安装流程，提供了一套便捷的安装脚本，Ubuntu系统上可以使用这套脚本安装：

    ```shell
    $ curl -fsSL get.docker.com -o get-docker.sh
    $ sudo sh get-docker.sh --mirror Aliyun
    ```

    #### 启动Docker CE

    ```shell
    $ sudo service docker start
    ```

    #### 建立docker用户组

    默认情况下，docker命令会使用Unix socket与Docker引擎通讯。而只有root用户和docker组的用户才可以访问Docker引擎的Unix socket。处于安全考虑，一般Linux系统上不会直接使用root用户。因此，最好的做法是将需要使用docker的用户加入docker用户组。

    建立docker组：

    ```shell
    $ sudo groupadd docker
    ```

    将当前用户加入docker组：

    ```shell
    $ sudo usermod -aG docker $USER
    ```

    退出当前终端并重新登录，进行如下测试。

    #### 测试docker是否安装正确

    ```shell
    $ docker run hello-world
    ```

* ### 使用Docker镜像
  ##### 从仓库获取镜像

  ```shell
  $	docker pull --help
  
  Usage:  docker pull [OPTIONS] NAME[:TAG|@DIGEST]
          #docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
  
  Pull an image or a repository from a registry
  
  Options:
    -a, --all-tags                Download all tagged images in the repository
        --disable-content-trust   Skip image verification (default true)
        --help                    Print usage
  ```

  例如获取Ubuntu18.04的镜像

  ```shell
  guanyin@software:~$ docker pull ubuntu
  Using default tag: latest
  latest: Pulling from library/ubuntu
  898c46f3b1a1: Pull complete
  63366dfa0a50: Pull complete
  041d4cd74a92: Pull complete
  6e1bee0f8701: Pull complete
  Digest: sha256:017eef0b616011647b269b5c65826e2e2ebddbe5d1f8c1e56b3599fb14fabec8
  Status: Downloaded newer image for ubuntu:latest
  ```

  上述没有添加具体的TAG将会获取最新版本的Ubuntu镜像，而在下载过程中可以看到是分层下载的，在下载结束会给出个文件镜像的sha256摘要，以确保下载一致性。

  ##### 运行该镜像

  ```shell
  docker run --help #将给出所有run相关的可选参数
  #目前我使用的建立jenkins容器的命令
  docker run -it --name jenkins -p 18080:8080 -p 18081:18081 -v /usr/work/guanyin:/home/ lava/ubuntu14.04-jdk8:jenkins-1.0.0
  ```

  解释一下参数：

  > - `-it`：这是两个参数，一个是 `-i`：交互式操作，一个是 `-t` 终端。我们这里打算进入 `bash` 执行一些命令并查看返回结果，因此我们需要交互式终端。
  > - `--name`: 指定容器的名字
  > - `-p`:映射端口到容器
  > - `-v`:上一节有说docker将是直接对宿主机器的文件系统进行读写，这里指定docker中的目录和宿主机的文件目录，格式为`宿主目录：docker内虚拟目录`
  > - `REPOSITORY:TAG`：指定要使用的仓库和TAG
  > - 其他命令
  > - `--rm`：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 `docker rm`。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 `--rm` 可以避免浪费空间
  > - `bash`：放在镜像名后的是**命令**，这里我们希望有个交互式 Shell，因此用的是 `bash`。

  #### 列出镜像

  ```shell
  guanyin@software:~$ docker images
  REPOSITORY        TAG                 IMAGE ID            CREATED             SIZE
  ubuntu            latest              94e814e2efa8        8 days ago          88.91 MB
  lava/ubuntu14.04-jdk8   jenkins-1.0.0 5d4dc5c28fea        10 months ago       3.386 GB
  bhaavan/battery-historian   latest    9a3a9fd0ca2f        2 years ago         921.6 MB
  lava/ubuntu14.04-jdk8    androidN-1.2.0   5684c90c9153     2 years ago        2.877 GB
  ```

  这里docker列出的占用体积大小是不准确的，docker使用的是Union FS，因此相同的层只需要保存一份即可，不同的容器可以共享，需要了解Union FS系统文件格式

  ##### 虚悬（dangling）镜像

  在镜像列表中可能会出现如下所示的镜像

  > ```bash
  > <none>       <none>              00285df0df87        5 days ago          342 MB
  > ```

  出现的原因：

  1. docker pull 了一个同名的镜像，之前的镜像名称被取消

  2. docker build 创建了一个已经存在的同名镜像，之前的镜像名称被取消

  ``` shell
  $ docker image -f dangling=true #显示当前的所有虚悬镜像
  $ docker image prune #这种镜像可以直接进行删除
  ```

  ##### 中间层镜像

  中间层镜像没有标签，这是docker在重复资源的过程中生成的，不需要删除。

  **在列出镜像时可以根据不同的需求添加参数，进行过滤显示**

``` shell
guanyin@software:~$ docker images --help

Usage:  docker images [OPTIONS] [REPOSITORY[:TAG]]

List images

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter value    Filter output based on conditions provided (default [])
      --format string   Pretty-print images using a Go template
      --help            Print usage
      --no-trunc        Don't truncate output
  -q, --quiet           Only show numeric IDs
```

##### 删除本地镜像

> docker rmi ubuntu #删除指定镜像

> docker rm container_name#移除指定容器

