---
layout: post
title: 基于Jenkins, Apache Mesos和Marathon的弹性高可用的持续集成环境（上篇）
tags: Jenkins Mesos CI scale
categories: scale CI
---

  持续集成(CI)是一种软件开发实践，使用得当，它会极大的提高软件开发效率并保障软件开发质量。Jenkins 是一个开源项目，提供了一种易于使用的持续集成系统；Mesos是Apache下的一个开源分布式资源管理框架，它被称为是分布式系统的内核；Marathon是Apache Mesos的一个管理长时应用(long-running applications)的framework，如果把Mesos比作数据中心kernel的话，那么Marathon就是init或者upstart的daemon。

  本文旨在探讨如何利用Jenkins，Apache Mesos和Marathon构建一套弹性的，高可用的持续集成环境。
  

为什么要把Jenkins运行到Apache Mesos上
=====================================

TODO

环境设置
========

  为了便于理解，这里我简化了Mesos/Marathon集群的架构，不再考虑集群本身的高可用性。至于如何利用zookeeper配置高可用的mesos/marathon集群，可以参考![Mesosphere的官方文档](https://mesos.apache.org/documentation/latest/mesos-architecture/)，这里不再展开。

  假设我有10个节点 ``192.168.3.1-192.168.3.10``，其中一个节点用作Marthon及Mesos-master，其它9个节点作为mesos的slave，如下所示。

    192.168.3.1  marathon/mesos-master
    192.168.3.2  mesos-slave
    192.168.3.3  mesos-slave
    ......
    192.168.3.10  mesos-slave
  

在Marathon上部署Jenkins的master实例
===================================

TODO

配置Jenkins Master实现弹性伸缩
==============================

TODO

利用git repo实现持久化保证系统高可用
==============================

TODO

总结
====

TODO


