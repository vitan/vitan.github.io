---
layout: post
title: Docker Research
tags: Docker LXC cgroup
categories: Linux
---


Docker是一个自动化分布式系统：大规模的Web部署、数据库集群、持续部署系统、面向服务的体系结构等。


简述
----
* * *
* * *

Docker是一个自动化分布式系统：大规模的Web部署、数据库集群、持续部署系统、面向服务的体系结构等。

Docker是一种增加了高级API的[LinuX Container（LXC）](https://linuxcontainers.org/)技术，提供了能够独立运行Unix进程的轻量级虚拟化解决方案。它提供了一种在安全、可重复的环境中自动部署软件的方式。

Docker是一个云计算平台，它利用Linux的LXC、AUFU、Go语言、cgroup实现了资源的独立，可以很轻松的实现文件、资源、网络等隔离，其最终的目标是实现类似PaaS平台的应用隔离。
 
使用Docker技术有望打破产品发布过程中应用开发工程师和系统工程师两者之间无法轻松协作发布产品的难题。这个Container容器技术可以把开发者从日常部署应用的繁杂工作中解脱出来，让开发者能专心写好程序；从系统工程师的角度来看也是一样，他们迫切需要从各种混乱的部署文档中解脱出来，让系统工程师专注在应用的水平扩展、稳定发布的解决方案上。


Docker要解决什么
---------------
* * *
* * *

Docker产生的目的就是为了解决以下问题:

###环境管理复杂

从各种OS到各种中间件再到各种App，一款产品能够成功发布，作为开发者需要关心的东西太多，且难于管理。Docker可以简化部署多种应用实例工作，比如Web应用、后台应用、数据库应用、大数据应用比如Hadoop集群、消息队列等等都可以打包成一个Image部署。

###云计算时代的到来

AWS的成功, 引导开发者将应用转移到云上, 解决了硬件管理的问题，然而软件配置和管理相关的问题依然存在。Docker的出现正好能帮助软件开发者开阔思路，尝试新的软件管理方法来解决这个问题。

###虚拟化手段的变化

云时代采用标配硬件来降低成本，采用虚拟化手段来满足用户按需分配的资源需求以及保证可用性和隔离性。然而无论是KVM还是Xen，在Docker看来都在浪费资源，因为用户需要的是高效运行环境而非OS, GuestOS既浪费资源又难于管理, 更加轻量级的LXC更加灵活和快速。

###LXC的便携性

LXC在 Linux 2.6 的 Kernel 里就已经存在了，但是其设计之初并非为云计算考虑的，缺少标准化的描述手段和容器的可便携性，决定其构建出的环境难于分发和标准化管理(相对于KVM之类image和snapshot的概念)。Docker就在这个问题上做出了实质性的创新方法。


Docker核心技术
-------------
* * *
* * *

Docker核心是一个操作系统级虚拟化方法, 理解起来可能并不像VM那样直观。

###隔离性: Linux Namespace(ns)

每个用户实例之间相互隔离, 互不影响。 一般的硬件虚拟化方法给出的方法是VM，而LXC给出的方法是container，更细一点讲就是kernel namespace。其中pid、net、ipc、mnt、uts、user等namespace将container的进程、网络、消息、文件系统、UTS("UNIX Time-sharing System")和用户空间隔离开。

####文件系统隔离：每个进程容器运行在一个完全独立的根文件系统里

类似chroot，将一个进程放到一个特定的目录执行。mnt namespace允许不同namespace的进程看到的文件结构不同，这样每个 namespace 中的进程所看到的文件目录就被隔离开了。同chroot不同，每个namespace中的container在/proc/mounts的信息只包含所在namespace的mount point。

####资源隔离：系统资源，像CPU和内存等可以分配到不同的容器中，使用cgroup

cgroups 实现了对资源的配额和度量。cgroups的使用非常简单，提供类似文件的接口，在/cgroup目录下新建一个文件夹即可新建一个group，在此文件夹中新建task文件，并将pid写入该文件，即可实现对该进程的资源控制。groups可以限制blkio、cpu、cpuacct、cpuset、devices、freezer、memory、net_cls、ns九大子系统的资源，以下是每个子系统的详细说明：

    1. blkio 这个子系统设置限制每个块设备的输入输出控制。例如:磁盘，光盘以及usb等等。
    2. cpu 这个子系统使用调度程序为cgroup任务提供cpu的访问。
    3. cpuacct 产生cgroup任务的cpu资源报告。
    4. cpuset 如果是多核心的cpu，这个子系统会为cgroup任务分配单独的cpu和内存。
    5. devices 允许或拒绝cgroup任务对设备的访问。
    6. freezer 暂停和恢复cgroup任务。
    7. memory 设置每个cgroup的内存限制以及产生内存资源报告。
    8. net_cls 标记每个网络包以供cgroup方便使用。
    9. ns 名称空间子系统。

以上九个子系统之间也存在着一定的关系.

####网络隔离：每个进程容器运行在自己的网络空间，虚拟接口和IP地址

不同用户的进程就是通过pid namespace隔离开的，有了pid namespace, 每个namespace中的pid能够相互隔离，但是网络端口还是共享host的端口。网络隔离是通过net namespace实现的， 每个net namespace有独立的network devices, IP addresses, IP routing tables, /proc/net目录。这样每个container的网络就能隔离开来。docker默认采用veth的方式将container中的虚拟网卡同host上的一个docker bridge: docker0连接在一起。

UTS("UNIX Time-sharing System") namespace允许每个container拥有独立的hostname和domain name, 使其在网络上可以被视作一个独立的节点而非Host上的一个进程。

####用户隔离： 每个进程容器可以有不同的user和group id

也就是说可以在container内部用container内部的用户执行程序而非Host上的用户。

###docker像git一样管理它的image

   AUFS (AnotherUnionFS) 是一种 Union FS, 简单来说就是支持将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)的文件系统, 更进一步的理解, AUFS支持为每一个成员目录(类似Git Branch)设定readonly、readwrite 和 whiteout-able 权限, 同时 AUFS 里有一个类似分层的概念, 对 readonly 权限的 branch 可以逻辑上进行修改(增量地, 不影响 readonly 部分的)。通常 Union FS 有两个用途, 一方面可以实现不借助 LVM、RAID 将多个disk挂到同一个目录下, 另一个更常用的就是将一个 readonly 的 branch 和一个 writeable 的 branch 联合在一起，Live CD正是基于此方法可以允许在 OS image 不变的基础上允许用户在其上进行一些写操作。Docker 在AUFS上构建的 container image也正是如此，接下来我们从启动 container 中的 linux 为例来介绍 docker 对AUFS特性的运用。

典型的启动Linux运行需要两个FS: bootfs + rootfs，bootfs (boot file system) 主要包含 bootloader 和 kernel, bootloader主要是引导加载kernel, 当boot成功后 kernel 被加载到内存中后 bootfs就被umount了. rootfs (root file system) 包含的就是典型 Linux 系统中的 /dev, /proc,/bin, /etc 等标准目录和文件。

对于不同的linux发行版, bootfs基本是一致的, 但rootfs会有差别, 因此不同的发行版可以公用bootfs，典型的Linux在启动后，首先将 rootfs 设置为 readonly, 进行一系列检查, 然后将其切换为 "readwrite" 供用户使用。在Docker中，初始化时也是将 rootfs 以readonly方式加载并检查，然而接下来利用 union mount 的方式将一个 readwrite 文件系统挂载在 readonly 的rootfs之上，并且允许再次将下层的 FS(file system) 设定为readonly 并且向上叠加, 这样一组readonly和一个writeable的结构构成一个container的运行时态, 每一个FS被称作一个FS层。![AUFS](http://infoqstatic.com/resource/articles/docker-core-technology-preview/zh/resources/0731016.png "AUFS")

得益于AUFS的特性, 每一个对readonly层文件/目录的修改都只会存在于上层的writeable层中。这样由于不存在竞争, 多个container可以共享readonly的FS层。 所以Docker将readonly的FS层称作 "image" - 对于container而言整个rootfs都是read-write的，但事实上所有的修改都写入最上层的writeable层中, image不保存用户状态，只用于模板、新建和复制使用。

上层的image依赖下层的image，因此Docker中把下层的image称作父image，没有父image的image称作base image。因此想要从一个image启动一个container，Docker会先加载这个image和依赖的父images以及base image，用户的进程运行在writeable的layer中。所有parent image中的数据信息以及 ID、网络和lxc管理的资源限制等具体container的配置，构成一个Docker概念上的container。![AUFS](http://infoqstatic.com/resource/articles/docker-core-technology-preview/zh/resources/0731018.png "AUFS")


###日志记录：Docker将会收集和记录每个进程容器的标准流（stdout/stderr/stdin），用于实时检索或批量检索

###变更管理：容器文件系统的变更可以提交到新的映像中，并可重复使用以创建更多的容器。无需使用模板或手动配置

###交互式shell：Docker可以分配一个虚拟终端并关联到任何容器的标准输入上，例如运行一个一次性交互shell。

Docker周边生态系统
----------------
* * *
* * *

###[Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes)

  最近 Google 开源了 Kubernetes ——一个 docker 集群的管理工具，自6月开源以来，受到各大厂商的关注，目前包括Microsoft，RedHat、Docker等企业都纷纷加入Kubernetes社区，这无疑让Kubernetes项目更为引人注目。**但是它现在仍然只能运行在 [GCE](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/gce.md) 上**。

  不同企业的研究方向分别如下：
  Microsoft 正在努力确保Kubernetes在Azure VMs的Linux环境中工作的更好。

  Red Hat 正在把Kubernetes带到开放的混合云。
  
  IBM 正在为Kubernetes和更广泛的Docker生态系统贡献代码，以确保容器是企业级的，并正与社区一起为该项目创建一个开放的治理模式。

  Docker 正在实现Kubernetes时间表上的所有的容器堆栈，并且开始考虑将关键性能上移以及让Kubernetes框架与Libswarm匹配。
 
  CoreOS 正在努力确保其分布式架构操作系统能够适用于Kubernetes。 
 
  Mesosphere 正积极的将Kubernetes和Mesos集成，使Kubernetes客户能够获得更加先进的调度和管理功能。
 
  SaltStack 正努力简化Kubernetes运行在其它环境下的部署流程。

###[Panamax](http://panamax.io/)

  Panamax被称为是一个“为人类而设计的Docker管理工具”。Panamax与其它的Docker集成工具不同的是它提供了一个基于Web的用户界面，在用户界面中，用户可以把多个Docker容器组合为模板并分享到GitHub。
  Panamax的最初版本运行在由Vagrant管理的VirtualBox上，由于Vagrant的限制，目前Panamax仅可运行在Mac和Linux的VirtualBox上，并不支持其他虚拟化平台。CenturyLink的云平台也将会支持Panamax。Panamax内部使用了CoreOS（一个为Docker优化的Linux发行版）以及与CoreOS相关的编配工具fleet、分布式key/value存储etcd。Panamax暂时只支持单机部署，不过Carlson承诺接下来将会支持多主机、多服务器。他说在敲定细节之前他们希望得到社区的帮助，fleet和etcd应该可以让架构的扩展相对容易。

###[CoreOS](https://coreos.com/)

  CoreOS是一个基于Linux，systemd和Docker的小型操作系统。允许快速的PXE boot。基于CoreOS构建的Docker Container在打包时有更大的灵活性，并且可以在毫秒内完成START。并且它还有一些有趣的[集群优势](https://coreos.com/blog/cluster-level-container-orchestration/)

###[Atomic](http://www.projectatomic.io/)

  Atomic是另一个小型的Linux操作系统，与CoreOS相比，Atomic的优势在于安全性和可靠性。它旨在实现 *SELinux* policy与Docker的交互，保护HOST的安全，同时保护Container之间的安全。这也是RHEL的一大优势。

总结
----
* * *
* * *

  网络上现在对于Docker都是正面评价，Google也声称其每天Start几百几千个Container，但是社区里仍然没有Docker在生产环境上的最佳实践。更多的是看好Docker的未来，以及开发者利用Docker所做的一些Image。
  从我个人看来，

  1. 我们可以利用Docker的隔离性构建自己纯净的开发环境。并与同事共享开发环境。
  2. 构建本地的PaaS平台来快速的熟悉分布式系统的一些基本配置。

  另外，在生产环境上，利用Docker部署图片，视频等静态文件CDN可能是一个不错的选择。

参考链接
-------

[Linux-Container-Runtime-Docker](http://www.csdn.net/article/2013-03-28/2814679-Linux-Container-Runtime-Docker)

[docker-core-technology-preview](http://www.infoq.com/cn/articles/docker-core-technology-preview)

[Panamax-launch](http://www.infoq.com/news/2014/08/panamax-launch)
