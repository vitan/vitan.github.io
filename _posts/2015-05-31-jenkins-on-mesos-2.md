---
layout: post
title: 基于Jenkins, Apache Mesos和Marathon的弹性高可用的持续集成环境（下篇）
tags: Jenkins Mesos CI auto-scaling
categories: scale CI
---

  在[基于Jenkins, Apache Mesos和Marathon的弹性高可用的持续集成环境（上篇）](http://vitan.github.io/scale/ci/2015/05/12/jenkins-on-mesos-1.html), 探讨了利用 marathon/Mesos 管理Jenkins集群的整个架构流程，以及如何一步步配置 Jenkins Master 来将其注册成为 Mesos 的 Framework 。本篇博客，我们主要解决Jenkins Master的数据持久化问题，marathon会在Jenkins Master因意外崩溃后重新部署其到某个mesos slave节点上，但marathon无法维护应用程序的数据，即我们需要一个 Jenkins Master 的数据持久化方法，由于Jenkins Master是将数据存储在XML文件而不是数据库中，这里可以利用jenkins插件[SCM Sync configuration plugin](https://wiki.jenkins-ci.org/display/JENKINS/SCM+Sync+configuration+plugin)来将Jenkins Master的数据同步到相应的repo。
