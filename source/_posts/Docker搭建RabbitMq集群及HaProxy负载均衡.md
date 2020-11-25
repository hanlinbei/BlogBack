---
title: Docker搭建RabbitMq集群及HaProxy负载均衡
categories:
  - 环境搭建
tags:
  - Docker
  - Mq
toc: true
abbrlink: 33196
date: 2020-06-30 11:11:57
---

使用同一台阿里云服务器搭建3个RabbitMq的集群和Haproxy负载均衡。
<!--more-->
## 创建rabbitmq容器

使用docker pull  rabbitmq拉取最新rabbimq镜像，docker pull haproxy 拉取haproxy镜像
{% asset_img 2020-07-06-18-41-37.png %}
创建docker网络 rabbtimanet 用于haproxy和rabbimq通信

```shell
docker network create rabbtimanet
```

{% asset_img 2020-07-06-18-43-06.png %}

创建三节点rabbitmq容器
rabbitmq1：

```shell
docker run -d --name=rabbitmq1 -p 5672:5672 -p 15672:15672 -e RABBITMQ_NODENAME=rabbitmq1 -e RABBITMQ_ERLANG_COOKIE='YZSDHWMFSMKEMBDHSGGZ'  -h rabbitmq1 --net=rabbtimanet rabbitmq:management
```

rabbitmq2：

```shell
docker run -d --name=rabbitmq1 -p 5673:5672 -p 15673:15672 -e RABBITMQ_NODENAME=rabbitmq2 -e RABBITMQ_ERLANG_COOKIE='YZSDHWMFSMKEMBDHSGGZ'  -h rabbitmq2 --net=rabbtimanet rabbitmq:management
```

rabbitmq3：

```shell
docker run -d --name=rabbitmq1 -p 5674:5672 -p 15674:15672 -e RABBITMQ_NODENAME=rabbitmq3 -e RABBITMQ_ERLANG_COOKIE='YZSDHWMFSMKEMBDHSGGZ'  -h rabbitmq3 --net=rabbtimanet rabbitmq:management
```

{% asset_img 2020-07-06-18-45-36.png %}

## rabbitmq集群

分别进入rabbitmq2 和rabbitmq3容器(docker exec -it 容器id /bin/bash)，执行以下：

```cmd
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbitmq1@rabbitmq1
rabbitmqctl start_app
```

{% asset_img 2020-07-06-18-46-24.png %}

## 部署Haproxy

编辑haproxy配置文件如下：

```shell
global
  daemon
  maxconn 256

defaults
  mode http
  timeout connect 5000ms
  timeout client 5000ms
  timeout server 5000ms

listen rabbitmq_cluster #监听5677端口转发到rabbitmq服务
  bind 0.0.0.0:5677
  option tcplog
  mode tcp
  balance leastconn
  server rabbit1 rabbitmq1:5672 check inter 2s rise 2 fall 3
  server rabbit2 rabbitmq2:5672 check inter 2s rise 2 fall 3
  server rabbit3 rabbitmq3:5672 check inter 2s rise 2 fall 3
listen http_front #haproxy的客户页面
  bind 0.0.0.0:80
  stats uri /haproxy?stats

listen rabbitmq_admin #监听8011端口转发到rabbitmq的客户端
  bind 0.0.0.0:8001
  server rabbit1 rabbitmq1:15672 check inter 2s rise 2 fall 3
  server rabbit2 rabbitmq2:15672 check inter 2s rise 2 fall 3
  server rabbit2 rabbitmq3:15672 check inter 2s rise 2 fall 3
```

创建haproxy容器

```shell
docker run -d --name rabbitmq-haproxy  -p 8090:80 -p 5677:5677 -p 8001:8001  --net=rabbtimanet -v /home/haproxy/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro haproxy:latest
```

通过外部8090访问haproxy容器的80端口，外部8001访问haproxy容器8001，外部5677访问haproxy 容器5677端口

## 测试

连接rabbitmq 的5677端口，并发送数据，检查haproxy的web页面，对每次的请求转发至不同的rabbitmq
{% asset_img 2020-07-06-18-48-54.png %}
通过haproxy的8001端口访问rabbitmq的客户端：
{% asset_img 2020-07-06-18-49-08.png %}
