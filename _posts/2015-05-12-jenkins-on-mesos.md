---
layout: post
title: 基于Jenkins, Apache Mesos和Marathon的弹性高可用的持续集成环境（上篇）
tags: Jenkins Mesos CI scale
categories: scale CI
---

持续集成(CI)是一种软件开发实践，使用得当，它会极大的提高软件开发效率并保障软件开发质量。Jenkins 是一个开源项目，提供了一种易于使用的持续集成系统；Mesos是Apache下的一个开源分布式资源管理框架，它被称为是分布式系统的内核；Marathon是Apache Mesos的一个管理长时应用(long-running applications)的framework，如果把Mesos比作数据中心kernel的话，那么Marathon就是init或者upstart的daemon。本文正是从持续集成的基本概念入手，通过具体实例，介绍了如何基于 Jenkins 快速搭建持续集成环境。
本文旨在探讨如何利用Jenkins，Apache Mesos和Marathon构建一套弹性的，高可用的持续集成环境。
