---
layout: post
title: 我在 CentOS 上使用 docker 与 Ubuntu 上性能有什么差别，或者说 Docker 使用 AUFS 与 DeviceMapper 作为存储驱动会有什么不同?
tags: docker
categories: docker
---

简单说，在某些场景下，DeviceMapper 的性能是要比 AUFS 差的，而在另一些场景下，反之。但无论是 DeviceMapper 还是 AUFS，亦或者是其他的 docker 存储驱动，它们严重影响到了容器性能是有前提条件的，那就是应用在频繁，或者说相对频繁的读写容器里面的文件系统。而这在实际的软件开发实践中，几乎是不可能，或者说完全可以规避掉的情况。首先，对于web 应用或者其他非存储型应用来说，它们一般会把数据存储到数据库中，所以，不会出现频繁读写容器里文件系统的情况；其次，如果应用需要频繁的读写本地文件系统，譬如把日志写到本地，或者Mysql存储到本地目录，对于这种情况，docker官方以及大家集体认同的方式是通过 docker -v 挂载数据盘来使用原生的系统文件驱动。从而这也就不会有什么太大了性能影响了。

  如果您想了解更多，请往下看，这里介绍下我整理的 docker 官方以及 Red Hat 公司开发者对于 AUFS，DeviceMapper 在不同场景下的性能对比。

  首先，AUFS 是基于 file-level 的存储驱动，即所有的容器都直接使用了宿主机的文件系统；而 DeviceMapper 是基于 block-level 的存储驱动，可以理解为每个容器有自己的文件系统，在block这一层做数据共享。DeviceMapper 存储驱动进一步又分为 loop-lvm 与 direct-lvm 两种方式，前一种我可以理解为 LiveCD 或虚拟VMware镜像加载系统，后一种理解为原生磁盘加载系统，所以后者是 docker 官方推荐的生产环境配置，而前者是在 CentOS 环境下安装 docker 的默认配置，这也是很多docker新人刚刚接触docker时吐槽 DeviceMapper 不给力的主要原因。
另外，AUFS 的性能缺点主要表现在应用第一次写上层 docker image layer 的文件时会有延迟，并且延迟随文件的增大，layer的层数增多而变大。 DeviceMapper 的性能缺点表现在docker daemon启动慢，docker容器启动相对较慢，频繁的小文件写入会有性能问题。
DeviceMapper 这些缺点中影响最大的是容器启动较慢的问题，但在配置了direct-lvm+volumn 的情况，创建并销毁1000个apache容器也只需要900s，这是 Red Hat 的开发者博客中详细测试过的[1]。
另外，附上docker官方对于 DeviceMapper 的优化建议[2]：
生产环境使用 direct-lvm 模式。即额外挂载数据盘作为docker的存储。
为你的docker存储配置ssd盘
将频繁读写的目录挂载到数据盘

[1] [http://developerblog.redhat.com/2014/09/30/overview-storage-scalability-docker/](http://developerblog.redhat.com/2014/09/30/overview-storage-scalability-docker/)
[2] [https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/](https://docs.docker.com/engine/userguide/storagedriver/device-mapper-driver/)
[3] [https://docs.docker.com/engine/userguide/storagedriver/aufs-driver/](https://docs.docker.com/engine/userguide/storagedriver/aufs-driver/)
[4] [http://www.infoq.com/cn/articles/analysis-of-docker-file-system-aufs-and-devicemapper#anch121685](http://www.infoq.com/cn/articles/analysis-of-docker-file-system-aufs-and-devicemapper#anch121685)
[5] [https://www.projectatomic.io/blog/2015/06/notes-on-fedora-centos-and-docker-storage-drivers/](https://www.projectatomic.io/blog/2015/06/notes-on-fedora-centos-and-docker-storage-drivers/)
