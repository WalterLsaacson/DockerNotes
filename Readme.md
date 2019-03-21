#### docker学习笔记

- docker 不同于其他虚拟化技术的关键点

  Docker 使用 Google 公司推出的 [Go 语言](https://golang.org/) 进行开发实现，基于 Linux 内核的 [cgroup](https://zh.wikipedia.org/wiki/Cgroups)，[namespace](https://en.wikipedia.org/wiki/Linux_namespaces)，以及[AUFS](https://en.wikipedia.org/wiki/Aufs) 类的 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 等技术，对进程进行封装隔离，属于 [操作系统层面的虚拟化技术](https://en.wikipedia.org/wiki/Operating-system-level_virtualization)。

  传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

分为如下部分

* [基本概念][file:///认识docker/md]
  * 了解镜像的组成架构
  * 如何获取镜像
  * 管理本地镜像
* 定制镜像
  * docker commit 定制事件
  * dockerfile构建实例
  * 其他的docker build用法
* Dockerfile指令举例和参考文档
* 走进UnionFS系统
* 操作容器
  * 