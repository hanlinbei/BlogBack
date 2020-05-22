---
title: Redis一主二从三哨兵
categories:
  - 环境搭建
tags:
  - 阿里云
  - 缓存
toc: true
abbrlink: 5310
date: 2020-05-16 18:12:57
---


本文简单讲述主从建立和哨兵的配置。本机测试redis为Redis-x64-3.2.100[Windows下载地址](https://github.com/MicrosoftArchive/redis/releases)，使用二台服务器作为测试。

<!--more-->
## 下载解压

下载完成后进行解压，然后复制两份作为从成员，构成一主二从。在主服务器上使用一份，从服务器上使用二份。
{% asset_img 2020-05-16-16-21-15.png %}
{% asset_img 2020-05-16-15-44-27.png %}

## 主从配置

### 主redis配置

编辑文件夹下redis.windows.conf文件，设置bind和port，因为要用到远程连接，所以绑定的ip 127.0.0.1要注释掉，否则远程是无法访问的。端口就使用默认的6379，注意要把阿里云后台的端口限制防火墙打开。
{% asset_img 2020-05-16-15-48-27.png %}

### 从redis配置

从redis配置：同样编辑文件夹下redis.windows.conf文件，设置bind和port。bind127.0.0.1同样注释掉，port分别是6380和6381。做完以上操作，如何标志这两redis是从关系呢？所以，还需要在配置文件中加上一行配置。注意：两个从redis都是相同的配置语句，因为都从属于同一个主redis。
{% asset_img 2020-05-16-15-51-05.png %}
由于我开启了密码登录验证，所以masterauth要添加上。

## 主从启动

在文件加中使用cmd进入命令窗口，输入redis-server redis.windows.conf即可启动（redis-server.exe和redis.windows.conf文件在相同文件夹下），依次启动主和从redis。

```shell
redis-server redis.windows.conf
```

{% asset_img 2020-05-16-15-53-11.png %}
{% asset_img 2020-05-16-15-56-08.png %}
通过命令查看各个redis的状态
{% asset_img 2020-05-16-16-00-18.png %}
slave 6380
{% asset_img 2020-05-16-16-01-09.png %}
slave 6381
{% asset_img 2020-05-16-16-01-39.png %}
当搭建好后主服务器是可读可写的，而从服务器是只读的。当主服务器宕机后，整个系统就瘫痪了，不能往从服务器写入数据，不能自动的把从服务器上升为主服务器。此时可通过哨兵模式来实现当主服务器宕机后，从服务器自动上升为主服务器。

## 哨兵模式配置

新建哨兵配置文件，分别命名为

```shell
sentinel.conf
sentinel2.conf
sentinel3.conf
```

哨兵配置文件内容
sentinel.conf

```shell
port 27000
#master
sentinel monitor master 39.100.107.169 6379 1
sentinel down-after-milliseconds master 5000
sentinel auth-pass master 123
sentinel config-epoch master 1
sentinel leader-epoch master 1
```

sentinel2.conf

```shell
port 27002
#slave 1
sentinel monitor master 39.100.107.169 6379 1
sentinel down-after-milliseconds master 5000
sentinel auth-pass master 123
sentinel config-epoch master 1
sentinel leader-epoch master 1
```

sentinel3.conf

```shell
port 27001
#slave 2
sentinel monitor master 39.100.107.169 6379 1
sentinel down-after-milliseconds master 5000
sentinel auth-pass master 123
sentinel config-epoch master 1
sentinel leader-epoch master 1
```

这里需要注意的是如果redis配置种添加了密码验证，一定要在sentinel文件里添加auth-pass这个参数，不然在从机上升为主机后，其他服务器连接不了。

哨兵配置文件说明

```shell
1. port :当前Sentinel服务运行的端口  
2.sentinel monitor mymaster 39.100.107.169 6379 1:Sentinel去监视一个名为mymaster的主redis实例，这个主实例的IP地址为本机地址39.100.107.169，端口号为6379，而将这个主实例判断为失效至少需要1个 Sentinel进程的同意，只要同意Sentinel的数量不达标，自动failover就不会执行  
3.sentinel down-after-milliseconds mymaster 5000:指定了Sentinel认为Redis实例已经失效所需的毫秒数。当 实例超过该时间没有返回PING，或者直接返回错误，那么Sentinel将这个实例标记为主观下线。只有一个 Sentinel进程将实例标记为主观下线并不一定会引起实例的自动故障迁移：只有在足够数量的Sentinel都将一个实例标记为主观下线之后，实例才会被标记为客观下线，这时自动故障迁移才会执行  
4.sentinel parallel-syncs mymaster 1：指定了在执行故障转移时，最多可以有多少个从Redis实例在同步新的主实例，在从Redis实例较多的情况下这个数字越小，同步的时间越长，完成故障转移所需的时间就越长  
5.sentinel failover-timeout mymaster 15000：如果在该时间（ms）内未能完成failover操作，则认为该failover失败  
```

### 哨兵测试

启动3个哨兵

```shell
redis-server.exe sentinel.conf --sentinel
redis-server.exe sentinel2.conf --sentinel
redis-server.exe sentinel3.conf --sentinel
```

{% asset_img 2020-05-16-16-25-19.png %}

### 测试主从切换

主机挂了后，从机是否能成功上位变为主机

先看下当前的redis状态

分别在客户端输入

```shell
info replication
```

{% asset_img 2020-05-16-16-14-53.png %}
当主机挂掉后，6381这个端口成了主机，这是通过哨兵的一个投票选择选出一个从机上升为主机。如果主机下次重新连接进来，那么它也不会立即成为主机，而是变为了从机。

## .NET Core中使用Redis集群

我们使用CSRedisCore来访问Redis，CSRedisCore是国内大牛开发的一个.net core redis 组件，源码可读性很强非常干净，几乎无任何依赖。性能相比ServiceStack.Redis和StackExchange.Redis会快10%左右，支持Redis的高级特性：订阅/发布，Pipeline，MGet/MSet，集群，分区。
创建一个.net core 控制台程序，然后添加nuget包

```shell
nuget Install-Package CSRedisCore
```

Program.cs代码

```c#
using System;
using System.Threading;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            //连接哨兵
            var csredis = new CSRedis.CSRedisClient("redis-master", new[] {"127.0.0.1:27000" });

            //初始化 RedisHelper
            RedisHelper.Initialization(csredis);

            while (true)
            {
                try
                {
                    Test();
                }
                catch(Exception ex)
                {
                    Console.WriteLine(ex.ToString());
                }
                Console.ReadLine();
            }
            Console.ReadKey();
        }

        static void Test()
        {
            RedisHelper.Set("name", "祝雷");//设置值。默认永不过期
            Console.WriteLine(RedisHelper.Get<String>("name"));

            RedisHelper.Set("time", DateTime.Now, 1);
            Console.WriteLine(RedisHelper.Get<DateTime>("time"));
            Console.WriteLine(RedisHelper.Get<DateTime>("time"));

            // 列表
            RedisHelper.RPush("list", "第一个元素");
            RedisHelper.RPush("list", "第二个元素");
            RedisHelper.LInsertBefore("list", "第二个元素", "我是新插入的第二个元素！");
            Console.WriteLine($"list的长度为{RedisHelper.LLen("list")}");
            Console.WriteLine($"list的第二个元素为{RedisHelper.LIndex("list", 1)}");
        }
    }
}
```

模拟故障进行测试，启动程序后，杀死主Redis进程，.net core程序再次访问Redis会出现一次异常检查，然后能正常切换到新的master上。

