---
layout: post
title: 基于Jenkins, Apache Mesos和Marathon的弹性高可用的持续集成环境（上篇）
tags: Jenkins Mesos CI scale
categories: scale CI
---

  持续集成(CI)是一种软件开发实践，使用得当，它会极大的提高软件开发效率并保障软件开发质量。Jenkins 是一个开源项目，提供了一种易于使用的持续集成系统；Mesos是Apache下的一个开源分布式资源管理框架，它被称为是分布式系统的内核；Marathon是Apache Mesos的一个管理长时应用(long-running applications)的framework，如果把Mesos比作数据中心kernel的话，那么Marathon就是init或者upstart的daemon。

  本文旨在探讨如何利用Jenkins，Apache Mesos和Marathon构建一套弹性的，高可用的持续集成环境。
  

###为什么要把Jenkins运行到Apache Mesos上

TODO

###环境设置

  为了便于理解，这里我简化了Mesos/Marathon集群的架构，不再考虑集群本身的高可用性。至于如何利用zookeeper配置高可用的mesos/marathon集群，可以参考[Mesosphere的官方文档](https://mesos.apache.org/documentation/latest/mesos-architecture/)，这里不再展开。

  假设我有40个节点 ``192.168.3.4-192.168.3.43``，其中一个节点用作Marthon及Mesos-master，其它39个节点作为mesos的slave，如下所示。

    192.168.3.4  marathon/mesos-master
    192.168.3.5  mesos-slave
    192.168.3.6  mesos-slave
    ......
    192.168.3.43  mesos-slave
  

###在Marathon上部署Jenkins的master实例

  Marathon支持web页面或者RESTapi两种方式发布应用，在``192.168.3.＊``内网执行下面的bash命令，就会通过Marathon的RESTapi在mesos slave上启动一个Jenkins master实例。

    git clone git@github.com:Dataman-Cloud/jenkins-on-mesos.git && cd jenkins-on-mesos && curl -v -X POST \
    -H 'Accept: application/json' \
    -H 'Accept-Encoding: gzip, deflate' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'User-Agent: HTTPie/0.8.0' \
    -d@marathon.json \
    http://192.168.3.4:8080/v2/apps

  *这里我在github上fork了[mesosphere的jenkins-on-mesos的repo](https://github.com/mesosphere/jenkins-on-mesos)到[DataMan-Cloud/jenkins-on-mesos](https://github.com/Dataman-Cloud/jenkins-on-mesos)，并进行了一些[改进](https://github.com/Dataman-Cloud/jenkins-on-mesos/commits?author=vitan)。*
  
  如果Jenkins master实例被成功部署，通过浏览器访问 ``[http://192.168.3.4:8080](http://192.168.3.4:8080)``(**请确定你的浏览器能够访问内网，如果不能，可以利用设置浏览器代理等方式来搞定**)可以在running tasks列表中找到 jenkins，点击进入详细信息页面，我们会看到下图：

  <img src="/assets/jenkins-master-on-marathon.png" style="width: 750px; height: 450px;" alt="Jenkins Master实例信息"/>

  访问``[http://192.168.3.4:5050/#/frameworks](http://192.168.3.4:5050/#/frameworks)``并在**Active Frameworks**中找到Marathon，点击进入详细信息页面，可以在该页面找到Jenkins Master具体运行到Mesos哪一台Slave上，如下图所示：

  <img src="/assets/jenkins-master-on-mesos-slave.png" style="width: 750px; height: 450px;" alt="Jenkins Master运行在mesos slave上"/>

  点击sandbox

  <img src="/assets/jenkins-master-on-mesos-slave-2.png" style="width: 750px; height: 300px;" alt="Jenkins Master运行在mesos slave上"/>

  另外，下图很好的解释了marathon framework是如何在Mesos上运行Jenkins Master的![Linked from ahunnargikar.files.wordpress.com临时占位](http://ahunnargikar.files.wordpress.com/2014/03/mesos3.png)

###配置Jenkins Master实现弹性伸缩

  接下来是通过配置Jenkins来让Jenkins注册成为Mesos的Framework。通过浏览器访问``[http://192.168.3.25:31052/](http://192.168.3.25:31052/)``,下面的截图是我通过Jenkins Master的Web UI逐步配置的过程。

  <img src="/assets/jenkins-home.png" style="width: 750px; height: 450px;" alt="Jenkins Master Home"/>
  <img src="/assets/jenkins-configure.png" style="width: 750px; height: 400px;" alt="Jenkins Master配置页面"/>
  <img src="/assets/jenkins-mesos-configure.png" style="width: 750px; height: 400px;" alt="Jenkins Master配置Mesos"/>

  如果Jenkins在Mesos上注册成功，访问``[http://192.168.3.4:5050/#/frameworks](http://192.168.3.4:5050/#/frameworks)``，我们可以找到jenkins Framework，如下图所示：

  <img src="/assets/jenkins-framework-on-mesos.png" style="width: 750px; height: 300px;" alt="Jenkins Framework on Mesos"/>

  现在我们可以同时启动多个构建作业来看一下Jenkins在Mesos上的弹性伸缩，在``http://192.168.3.25:31052/``上新建一个名为``test``的工程，通过设置``Label Expression``为``mesos``来限制该工程在特定的slave上运行，并配置其构建过程为运行一个shell命令``top``，如下图所示：

  <img src="/assets/test-job-config.png" style="width: 750px; height: 450px;" alt="配置构建作业"/>

  把该工程复制3份``test2``、``test3``和``test4``，并同时启动这4个工程的构建作业，Jenkins Master就会向Mesos申请资源，如果资源分配成功，Jenkins Master就在获得的slave节点上进行作业构建，如下图所示：

  <img src="/assets/building-jobs.png" style="width: 750px; height: 340px;" alt="构建作业列表"/>

因为在前面的系统配置里我们设置了*执行者数量*为2（即最多有两个作业同时进行构建），所以在上图中我们看到两个正在进行构建的作业，而另外两个作业在排队等待。
  
####配置Jenkins Slave参数(可选)

  在使用Jenkins进行项目构建时，我们经常会面临这样一种情形，不同的作业对slave的性能有不同的要求，有些作业需要在配置很高的slave机器上运行，但是有些则不需要。为了提高资源利用率，显然，我们需要一种手段来向不同的作业分配不同的资源。通过设置Jenkins Mesos Cloud插件的slave info，我们可以很容易的满足上述要求。 具体的配置如下图所示：

  <img src="/assets/jenkins-config-slave.png" style="width: 750px; height: 450px;" alt="Jenkins 配置 slave"/>

###利用git repo实现持久化保证系统高可用

TODO

基于Marathon 0.8? Mesos-version, Jenkins Version

###总结

TODO

###参考

[delivering-ebays-ci-solution-with-apache-mesos](http://www.ebaytechblog.com/2014/05/12/delivering-ebays-ci-solution-with-apache-mesos-part-ii/)
