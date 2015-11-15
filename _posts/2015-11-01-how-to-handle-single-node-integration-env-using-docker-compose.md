---
layout: post
title: 使用 docker-compose 进行单机微服务开发环境集成的实践
tags: docker-compose micro-service integration-testing
categories: docker best-practice
---

最近在做一个云管理平台，产品初期我们就已经决定使用 docker 容器技术来进行开发环境隔离，以及产品的持续交付部署等，同时也参考了微服务架构的思想，将产品拆成了各个微服务模块。其中涉及到的模块除了标准的 MySQL, Nginx, Redis, RabbitMQ 外还包括了团队开发的 auth, cluster, app, metrics, agent，streaming 和 frontend 等数个模块,  摊子铺得有点儿大。并且不同的模块团队成员在使用 Python，Golang，JS 等不同的开发语言， 随之带来的一个问题就是开发人员如何快速高效的在本地机器将这一套环境搭起来，毕竟要求每个工程师熟悉产品的所有模块的配置是不现实，另外对于前端工程师来说，他们也没有必要花精力在后台的搭建上。

为了解决这个问题，前期我尝试通过 `Makefile`，Dockerfile 和各种脚本文件将这些模块组织起来，使用了一段时间后，感觉维护起来非常吃力，无法完全自动化的将一套环境配置起来, 仍然需要人工介入。 后来就开始尝试使用 docker-compose 来将这些模块编排到一块儿，一路下来感觉维护成本小了很多。 一个 `docker-compose.yml` 就将好多脚本工作替掉了。故撰文分享之。


## 代码结构

下面是深度为3的代码组织树，我只保留了对本文有用的信息。

```tree
.
├── Changes.md
├── Makefile
├── README.md
├── ReleaseNote.md
├── bin
├── docker-compose.yml
├── docs
│   ├── userguide.md
│   └── prd.md
├── frontend
│   ├── glance
│   │   ├── README.md
│   └── conf
│   │   ├── nginx-glance.conf
│   │   └── nginx-market.conf
│   └── market
│       └── README.md
└── omega
    ├── app
    │   ├── Dockerfile
    │   └── README.md
    ├── auth
    │   ├── Dockerfile
    │   └── README.md
    └── cluster
        ├── Dockerfile
        └── README.md
```

其中目录

- frontend 里面包含两个子模块 glance 和 market ， 是两个独立的前端模块， 分别用 nginx 部署。
- omega 里面包含3个后台子模块 app , auth 和 cluster ， 这三个子模块有各自的 Dockerfile， 前端的 http 请求会通过 nginx 转发到相应后台模块的 RESTful 接口。 另外，后台子模块之间通过 RabbitMQ 通信。


## docker-compose.yml

首先这里给出部分 ``docker-compose.yml`` 的内容

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
    - auth
    - app
market:
  image: nginx:1.9.6
  ports:
    - "8001:80"
  links:
    - cluster
    - auth
    - app
  volumes:
    - ./frontend/market:/usr/share/nginx/html:ro
    - ./frontend/conf/nginx-market.conf:/etc/nginx/nginx.conf:ro
cluster:
  build: ./omega/cluster
  volumes:
    - ./omega/cluster:/code
  links:
    - mysql
    - redis
    - rmq
auth:
  build: ./omega/auth
  volumes:
    - ./omega/auth:/code
  links:
    - mysql
    - redis
    - rmq
app:
  build: ./omega/app
  volumes:
    - ./omega/app:/code
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

其实我们可以从 `docker-compose.yml` 中很直观的看出代码组织结构。在repo 的根目录执行命令：

```bash
docker-compose -p omega up -d
```

会将各个 container 以deamon 形式启动。

```bash
root#docker ps
CONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS              PORTS                           NAMES
6dc9c3a9afd0        nginx:1.9.6                     "nginx -g 'daemon off"   2 days ago          Up 2 days           80/tcp, 0.0.0.0:8001->443/tcp   omega_market_1
7ff7e7dbd905        nginx:1.9.6                     "nginx -g 'daemon off"   2 days ago          Up 2 days           80/tcp, 0.0.0.0:8000->443/tcp   omega_glance_1
774002924ae7        omega_cluster                   "/bin/sh -c './run.sh"   2 days ago          Up 2 days                                           omega_cluster_1
774002924ae7        omega_auth                      "/bin/sh -c './run.sh"   2 days ago          Up 2 days                                           omega_auth_1
774002924ae7        omega_app                       "/bin/sh -c './run.sh"   2 days ago          Up 2 days                                           omega_app_1
8d5bbefc85a4        redis:3.0.5                     "/entrypoint.sh redis"   2 days ago          Up 2 days           6379/tcp                        omega_redis_1
c34d7733e690        mysql:5.6                       "/entrypoint.sh mysql"   2 days ago          Up 2 days           3306/tcp                        omega_mysql_1
bca05fd1abac        rabbitmq:3.5.6                  "/docker-entrypoint.s"   2 days ago          Up 2 days           4369/tcp, 5672/tcp, 25672/tcp   omega_rmq_1
```

至此， 只要 nginx 配置没有问题， 我就可以通过浏览器访问 containers 所在机器的 `8001`，`8000` 端口来使用 market, 或者 glance 服务了。

## 服务发现

玩转 docker-compose 很重要的一点就是 container 之间的发现， 利用好这点，我们可以节省很多配置时间。

### Nginx 是如何发现后台模块的

下面是 nginx-glance.conf 的部分内容

```nginx.conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

http {
    index    index.html;
    server {
        listen       80;
        server_name  glance;
        root    /usr/share/nginx/html;

        location / {
          try_files $uri /index.html;
        }
        location ~ /api/v1/cluster {
          auth_request    /_auth;
          proxy_pass      http://cluster:8888/api/v1/cluster;
        }
        location ~ /api/v1/app {
          auth_request    /_auth;
          proxy_pass      http://app:8888/api/v1/app;
        }
        location = /_auth {
          internal;
          proxy_pass http://auth:8888/api/v1/auth;
          proxy_pass_request_body off;
          proxy_set_header Content-Length "";
          proxy_set_header X-Original-URI $request_uri;
        }
    }
}
```

简单说， 在容器 `omega-glance` 中是可以通过 `auth`, `app`, `cluster` 分别发现容器 `omega-auth`, `omega-app`, `omega-cluster` 的。 `docker-compose` 通过 `/etc/hosts` 的设置保证了这一点。 这样，在任何单机环境中，我们的 nginx 配置文件无须变更。

同理， 我们的 `app`, `auth`, `cluster` 模块也是通过这种方式访问 `mysql`, `rmq` 和 `redis` 服务的， 即，在单机开发环境中，我们的后台模块的配置文件也无需变更。


## 提高开发速度

假设我是 cluster 模块的开发， 为了在当前这套结构中快速看到我对 cluster 模块的更改，我通过挂载外部代码目录的方式来解决。下面是 cluster 模块 Dockerfile 的部分内容：

```Dockerfile
FROM python:3.4

MAINTAINER weitao wtzhou@dataman-inc.com

ADD . /code
WORKDIR /code

RUN pip3 install -r omega/cluster/requirements.txt
RUN mkdir /var/log/omega
CMD ./run.sh create_db; ./run.sh
```

另外，我已经在 docker-compose.yml 中声明了将 volumes ./omega/cluster 挂载到容器中的 /code 目录, 同时，由于我的 cluster 模块本身已经支持运行时检测代码改变并自动重启, 所以在开发过程中，我只需将改变保存，立刻就可以测试这些改变。

## 一个小坑

由于 docker 没有 `wait` 机制，即我们在启动容器时无法确定这个容器是在启动中还是已经完全初始化完毕。这也是 docker-compose 目前还未解决的 issue [Is there a way to delay container startup to support dependant services with a longer startup time](https://github.com/docker/compose/issues/374)。对应到我们这里， 由于 mysql 容器启动时需要一段时间，而容器 cluster，auth, app 模块又需要在启动后与 mysql 发生连接，这就可能导致cluster等容器连接一个启动中的MySQL，从而导致启动失败。 我们目前的解决方案是在 cluster，app，auth模块中实现MySQL重连机制，这样，我们可以不停的尝试重连MySQL直到启动成功。

## 优雅的退出

最后，由于所有的环境都live在容器中，我们可以很方便的推出整个环境，不留一丝残留，只需在终端执行下述命令即可，

```bash
    docker-compose -p omega stop
    docker-compose -p omega rm -f
```
