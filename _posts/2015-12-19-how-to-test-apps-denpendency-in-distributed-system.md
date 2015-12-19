---
layout: post
title: how to test apps denpendency in distributed system
tags: nginx
categories: distributed system
---

https://github.com/vitan/nginx-log-request-body

# nginx-log-request-body

nginx-log-request-body is logging the request_body , which is useful to me on debugging distributed apps deploying different servers. By making the apps post data to nginx-log-request-body server, I can debug sth.

## how to use it

1. start nginx container and echo the log forcely

```bash
docker logs -f `docker run -p 80:80 -d 2breakfast/nginx-log-request-body:latest`
```

2. post the test data to the nginx server
```bash
curl -X POST http://localhost:80/post -d 'test'
```
