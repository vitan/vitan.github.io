---
layout: post
title: 配置 git 使得本地无缝 push/pull 跳板机后内网的 repo 
tags: ssh git
categories: git ssh
---

  单刀直入，如果公司选择公有云作为开发/生产环境，我们的代码库譬如 gitlab， 往往会部署到内网的某台 node 上，同时，我们又想在本地自己的笔记本上进行开发，大致架构如下图所示。

  <img src="/assets/git-push-intranet.png" style="width: 550px; height: 200px;" alt="开发环境架构"/>

  由于只有跳板机绑定了公网，其他的机器都在跳板机后面的子网内，需要跳板机进行转发。显然，我们本地的开发机需经由跳板机才能 access 内网（跳板机后面）的 gitlab server。那么我们可以对本地的 git 以及 ssh 进行一些配置，使本地无缝的 push/pull 内网的 repo。
  
  首先假设跳板机的公网 IP 为 `10.1.2.3`, gitlab server 的内网 IP 为 `192.168.3.1`, 本地使用 `ubuntu` OS.
  
  1. 配置 ssh config `~/.ssh/config`
      这是我的 `ssh config` 部分内容，将 *HostName*, *User*, *IdentityFile* 相应的替换成你自己的。

      ```bash
      ~ » cat .ssh/config
        Host dataman-route
            HostName 10.1.2.3
            User wtzhou
            IdentityFile ~/.ssh/id_dataman_rsa
        Host 192.168.*.*
            User wtzhou
            IdentityFile ~/.ssh/id_dataman_rsa
            ProxyCommand nc -x localhost:10000 %h %p
      ```

  2. 打开本地 `ssh tunnel`
  3. 现在就可以 `git clone` 我们内网 gitlab 上的 repo 了, 譬如clone项目glance
