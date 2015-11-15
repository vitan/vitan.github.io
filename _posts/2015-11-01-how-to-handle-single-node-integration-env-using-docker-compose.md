---
layout: post
title: 使用 docker-compose 进行单机微服务开发环境集成的实践
tags: docker-compose micro-service integration-testing
categories: docker best-practice
---

最近在做一个云管理平台，产品初期我们就已经决定使用 docker 容器技术来进行开发环境隔离，以及产品的持续交付部署等，同时也参考了微服务架构的思想，将产品拆成了各个微服务模块。其中涉及到的模块除了标准的 MySQL, Nginx, Redis, RabbitMQ 外还包括了团队开发的 auth, cluster, app, metrics, agent，streaming 和 frontend 等数个模块,  摊子铺得有点儿大。并且不同的模块团队成员在使用 Python，Golang，JS 等不同的开发语言， 随之带来的一个问题就是开发人员如何快速高效的在本地机器将这一套环境搭起来，毕竟要求每个工程师熟悉产品的所有模块的配置是不现实，另外对于前端工程师来说，他们也没有必要花精力在后台的搭建上。

为了解决这个问题，前期我尝试通过 `Makefile`，Dockerfile 和各种脚本文件将这些模块组织起来，使用了一段时间后，感觉维护起来非常吃力，无法完全自动化的将一套环境配置起来, 仍然需要人工介入。 后来就开始尝试使用 docker-compose 来将这些模块编排到一块儿，一路下来感觉维护成本小了很多。 一个 `docker-compose.yml` 就将好多脚本工作替掉了。故撰文分享之。


首先这里给出我的 ``docker-compose.yml``

```yml
glance:
  image: nginx:1.9.6
  ports:
    - "8000:80"
  volumes:
    - ./frontend/glance:/usr/share/nginx/html:ro
    - ./frontend/conf/nginx-glance.conf:/etc/nginx/nginx.conf:ro
  links:
    - cluster
market:
  image: nginx:1.9.6
  ports:
    - "8001:80"
  links:
    - cluster
  volumes:
    - ./frontend/market:/usr/share/nginx/html:ro
    - ./frontend/conf/nginx-market.conf:/etc/nginx/nginx.conf:ro
cluster:
  build: ./omega
  volumes:
    - ./omega:/code
  links:
    - mysql
    - redis
    - rmq
redis:
  image: redis:3.0.5
  command: redis-server --appendonly yes
rmq:
  image: rabbitmq:3.5.6
  environment:
    - RABBITMQ_DEFAULT_USER=guest
    - RABBITMQ_DEFAULT_PASS=guest
mysql:
  image: mysql:5.6
  volumes:
    - ./omega/omega/mysql_settings/my.cnf:/etc/my.cnf
  environment:
    - MYSQL_ROOT_PASSWORD=rootpassword
```
