---
layout: post
title: 基于Jenkins, Apache Mesos和Marathon的弹性高可用的持续集成环境（下篇）
tags: Jenkins Mesos CI auto-scaling
categories: scale CI
---

  在[基于Jenkins, Apache Mesos和Marathon的弹性高可用的持续集成环境（上篇）](http://vitan.github.io/scale/ci/2015/05/12/jenkins-on-mesos-1.html), 探讨了利用 marathon/Mesos 管理Jenkins集群的整个架构流程，以及如何一步步配置 Jenkins Master 来将其注册成为 Mesos 的 Framework 。本篇博客，我们主要解决Jenkins Master的数据持久化问题，marathon会在Jenkins Master因意外崩溃后重新部署其到某个mesos slave节点上，但marathon无法维护应用程序的数据，即我们需要一个 Jenkins Master 的数据持久化方法，由于Jenkins Master是将数据存储在XML文件而不是数据库中，这里可以利用jenkins插件[SCM Sync configuration plugin](https://wiki.jenkins-ci.org/display/JENKINS/SCM+Sync+configuration+plugin)来将Jenkins Master的数据同步到相应的repo。

###在内部的代码库或者 github 上创建一个 git repo

  我们需要创建一个名为 **jenkins-on-mesos** 的 gitrepo ， 譬如：**git@gitlab.dataman.io:wtzhou/jenkins-on-mesos.git** 。 这个 repo 是 jenkins 插件 [SCM Sync configuration plugin](https://wiki.jenkins-ci.org/display/JENKINS/SCM+Sync+configuration+plugin) 用来同步jenkins数据的。

  另外，对于SCM-Sync-Configuration来说，非常关键的一步是保证其有权限 pull/push 上面的gitrepo。 以我们公司的内部环境为例， 在mesos集群搭建时，我们首先使用ansible为所有的mesos slave节点添加了用户**core**并生成了相同的**ssh keypair**，同时在内部的gitlab上注册了用户**core**并上传其在slave节点上的公钥，然后添加该用户**core**为repo **git@gitlab.dataman.io:wtzhou/jenkins-on-mesos.git**的*developer*或者*owner*，这样每个mesos slave节点都可以以用户**core**来 pull/push 所给的gitrepo了。
