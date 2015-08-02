---
layout: post
title: 如何配置 git 使得本地可以 push/pull 跳板机后内网的 repo 
tags: ssh git
categories: git ssh
---

  单刀直入，如果公司选择公有云作为开发/生产环境，我们的代码库譬如 gitlab， 往往会部署到内网的某台 node 上，同时，我们又想在本地自己的笔记本上进行开发，大致架构如下图所示。

  <img src="/assets/git-push-intranet.png" style="width: 630px; height: 200px;" alt="开发环境架构"/>

  由于只有跳板机绑定了公网，其他的机器都在跳板机后面的子网内，需要跳板机进行转发。显然，我们本地的开发机需经由跳板机才能 access 内网（跳板机后面）的 gitlab server。
