---
layout: post
title:  关于高并发、大数据
date:   2017-08-11 00:00:00 +0800
categories: document
tag: 并发
---

* content
{:toc}

# 关于高并发、大数据

## 网站的“大小”

&emsp;&emsp;谈论一个网站的热度，总免不了会询问日均PV、同时在线人数、注册用户数等运营数据，但从技术角度来看，这几个数值没有一个可以放在一起比较的——一个静态网站的PV跟一个SNS类/Web Game网站的PV根本就不是一回事。这里，我们从另一个角度——并发来谈谈Web网站的数量级。

大致根据理论最大QPS（Quest Per Second每秒请求数），给网站做几个分类：  
<b>50QPS一下——小网站</b>  
&emsp;&emsp;简单的小网站。

<b>50~100QPS——DB极限型</b>  
&emsp;&emsp;大部分的关系型数据库的每次请求大多都能控制在0.01秒左右，即便你的网站每页面只有一次DB请求，那么页面请求无法保证在1秒内完成100个请求，这个阶段要考虑做Cache或者多DB负载。无论那种方案，网站重构是不可避免的。

<b>300~800QPS——带宽极限型</b>  
&emsp;&emsp;目前服务器大多用了IDC提供的“百兆带宽”，这意味这网站出口的实际带宽是8M Byte左右。假定每个页面只有10K Byte，在这个并发条件下，百兆带宽已经吃完了。首要考虑是CDN加速/异地缓存，多机负载等技术。

<b>500~1000QPS——内网带宽极限+Memcache极限型</b>  
&emsp;&emsp;由于key/value的特性，每个页面对Memcache的请求远大于直接对DB的请求，Memcache的悲观并发数在2w左右，看似很高，但事实上大多数情况下，首先是有可能在此之前内网的带宽就已经吃完，接着是在8K QPS左右的情况下，Memcache已经表现出了不稳定，如果代码上没有足够的优化，可能直接将压力转嫁到了DB层上，这就最终导致整个系统在达到某个阀值之上，性能迅速下滑。  

<b>1000~2000QPS——Fork/Select，锁模式极限型</b>  
&emsp;&emsp;线程模型决定吞吐量。不管你系统中最常见的锁是什么锁，这个级别下，文件系统访问锁都成为了灾难。这就要求系统中不能存在中央节点，所有的数据都必须分布存储，数据需要分布处理。总之，分布。

<b>2000QSP以上——C10K极限</b>  
&emsp;&emsp;尽管现在很多应用已经实现了C25K，但短板理论告诉我们，决定网站整体并发的永远是最低效的那个环节。

## 针对网站优化  
1、对常用功能建立缓存模块  
2、页面静态化  
3、独立图片服务器、文件服务器  
4、数据库集群   
5、负载均衡  
6、CDN  

&emsp;&emsp;针对高并发、大数据，我们的主体思想是：<font color="darkblue">“分而治之，多级分流”</font>。我们可以在浏览器（或API）端、服务器前端、中间层、数据库端分别采取相应的措施。  

## 1、浏览器（API）端  
&emsp;&emsp;优化的点：资源、访问速度  

### 1.1、资源：js、css、页面文件、图片等。
> 优化：  
a、资源静态化；  
b、图片服务器、文件服务器；  
c、资源压缩；

### 1.2、访问速度
> 优化：  
a、CDN加速；  
b、优化代码处理逻辑，提高响应速度；  

## 2、服务器前端  

&emsp;&emsp;优化的点：分布式、服务化、静态化、安全性

### 2.1、分布式  
> 优化：  
a、Tomcat采用apr/nio模式增加吞吐量；  
b、负载均衡 tomcat + nginx；  
c、tomcat集群；  
d、ZooKeeper；  
e、分布式缓存；  

&emsp;&emsp;相关问题：   
1、登录问题 SSO；  
2、内存共享，如session；  
3、跨域访问；  

### 2.2、服务化
> 优化：  
a、远程调用和服务治理 dubbo；  
b、服务层分割；  

### 2.3、静态化  
> 优化：  
a、页面静态化；  
b、动静分离；  

### 2.4、安全性  
> 优化：
a、

## 3、中间层  
&emsp;&emsp;优化的点：缓存、消息队列

> 优化：  
a、缓存 Redis、Mencache
b、消息队列 ActiveMQ

## 4、数据库端
&emsp;&emsp;优化的点：分割、集群

### 4.1、分割
> 优化：  
a、主从分离；  
b、读写分离；  
c、分库分表；  
d、数据垂直和水平切分；  

### 4.2、集群

## 5、其他
> 优化：  
a、JVM调优；  
b、数据一致性：ZooKeeper    
c、事务处理：数据库事务、分布式事务；  
