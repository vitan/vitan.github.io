---
layout: post
title: Mesos 持久化方案初探
tags: Mesos
categories: cluster
---

  如何将 Mysql, Mongodb 等存储型服务或有状态服务融到 Mesos 里一直是大家在使用 Mesos 时纠结头疼的问题。我们认为只有让 MySql 集群，MongoDB 集群，RabbitMQ 集群等等所有这些都使用 Mesos 的资源池，Mesos 才能称之为一个真正的分布式操作系统，才能提高整个分布式环境的资源利用率。然而，在 mesos 0.22 之前，社区并没有给出一个 generic 的解决方案，有的只是某些团队针对个例开源的项目，譬如，twitter 开源的 [mysos](https://github.com/twitter/mysos)，一个运行 Mysql 实例的 mesos framework。抛开 mysos 等的可用性不谈，我认为 mesos 需要的是一个统一的，公用的解决方案。最近 mesos 社区在其邮件列表里公布了[Mesos 0.23 发布计划概要 (Proposal)](http://mail-archives.apache.org/mod_mbox/mesos-user/201506.mbox/%3CCAK8jAgMWjt=wp9hMpQ9mAqTCnP3hCu6M7XYvkdov4NfyWf=X8g@mail.gmail.com%3E), 其提到了3个重要的feature：SSL, Persistent Volumes 和 Dynamic Reservations, 后两者都是跟存储相关的功能。 在我看来，这些功能完美解决了 mesos 的持久化问题。 这里我会分享下 mesos 解决此问题的大致思路。另外，由于本篇 blog 写作时，mesos 0.23 还没有 release，所以可能跟最终版本有些出入。而且，从 Mesos 的原始 Paper 可以看到，Mesos 是为 short-term 或者无状态的计算型任务而生的，所以说使用 Mesos 来管理 long-term 的服务，其稳定性和可行性，还有待实践验证。

###需要在 Mesos 上持久化的服务都会成为 Mesos 的 Framework

reserved role
划定一批slave来做持久化

###磁盘，cpu，内存都需要区分持久化和非持久化部分

###磁盘隔离

生产环境挂载disk

du poll或者kernel event

###动态的 slave 属性，framework可编辑
