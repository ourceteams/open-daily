---
title: hotkey
date: 2021-10-24 17:23:02
tags:
---

大家好，我是爱撸码的开源大叔！

双十一时候，各大电商的流量都是很大的，过年时候火车票也都是秒光，这些流量是可以提前预测的，可以提前加服务器，还有些流量无法提前预测，像微博就承受了太多压力，饭圈突然就来一个热点新闻，比如xxx pc被抓、xxx宣布离婚、xxx公布恋情。。。大家都懂的



![](https://server.xmyeditor.com/picture/19/786e6392337e3229ba358f3c1d29b4c6.jpg)

互联网会有突发性无法预先感知的热点数据，大量请求同一个商品、恶意爬虫、海量请求同一个接口，大量的请求进来，我们需要考虑的点就很多了，**缓存雪崩**，**缓存击穿**，**缓存穿透**这些都是有可能发生的，甚至DB挂了，那就很难受了，最后背锅的还是**开发**。

今天大叔推荐的hotkey就可以解决这个问题。APP后台热数据探测框架，对任意突发性的无法预先感知的热点数据，推送到所有服务端JVM内存中，以大幅减轻对后端数据存储层的冲击，并可以由使用者决定如何分配、使用这些热key（譬如对热商品做本地缓存、对热用户进行拒绝访问、对热接口进行熔断或返回默认值）。这些热数据在整个服务端集群内保持一致性，并且业务隔离，worker端性能强悍。

### 系统架构



![](https://images.gitee.com/uploads/images/2020/0616/105737_e5b876cd_303698.png)

**1、etcd集群**

etcd作为一个高性能的配置中心，可以以极小的资源占用，提供高效的监听订阅服务。主要用于存放规则配置，各worker的ip地址，以及探测出的热key、手工添加的热key等。

**2、client端jar包**

就是在服务中添加的引用jar，引入后，就可以以便捷的方式去判断某key是否热key。同时，该jar完成了key上报、监听etcd里的rule变化、worker信息变化、热key变化，对热key进行本地caffeine缓存等。

**3、worker端集群**

worker端是一个独立部署的Java程序，启动后会连接etcd，并定期上报自己的ip信息，供client端获取地址并进行长连接。之后，主要就是对各个client发来的待测key进行累加计算，当达到etcd里设定的rule阈值后，将热key推送到各个client。

**4、dashboard控制台**

控制台是一个带可视化界面的Java程序，也是连接到etcd，之后在控制台设置各个APP的key规则，譬如2秒出现20次算热key。然后当worker探测出来热key后，会将key发往etcd，dashboard也会监听热key信息，进行入库保存记录。同时，dashboard也可以手工添加、删除热key，供各个client端监听。

### 适用场景

- mysql热数据本地缓存
-  redis热数据本地缓存
- 黑名单用户本地缓存
-  爬虫用户限流 接口、用户维度限流
- 单机接口、用户维度限流
- 集群用户维度限流
- 集群接口维度限流

### 工作流程

1、搭建搭建etcd集群

可以理解成微服务中注册中心

2、启动dashboard配置规则

![](https://images.gitee.com/uploads/images/2020/0622/175255_e1b05b4c_303698.png)

3、启动worker集群

worker与client进行长连接，计算各个client发来的key，当某key达到规则里设定的阈值后，将其推送到该APP全部client。

4、启动client端

client与worker建立长连接，定时把待测key到对应的worker。当worker探测出来热key后，会推送回来，client里提供了方法，根据业务做相应的处理，比如限流、降级访问的部分接口等等。

### 性能表现

每10秒打印一行，totalDealCount代表处理过的key总量，可以看到每10秒处理量在270万-310万之间，对应每秒30万左右QPS。

![](https://images.gitee.com/uploads/images/2020/0611/152336_78597937_303698.png)

![](https://images.gitee.com/uploads/images/2020/0611/152249_4ac01178_303698.png)

经历了多次大促压测演练以及618大促、双11大促，表现相当稳定。

### 总结

大叔总结一下，这个项目的工作原理，client和worker都会注册到etcd集群中，在dashboard控制台中配置规则，client定时把key上报给worker集群，worker计算client发来的key，当达到系统设置的阈值，worker把key推送给全部的client，client做相应的业务处理。

听了大叔的介绍，各位小伙伴有没有心动？心动不如行动，赶紧去公众号后台回复「**hotkey**」，获取开源项目的地址吧~~~

