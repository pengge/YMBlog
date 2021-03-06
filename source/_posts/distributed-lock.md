---
title: 分布式锁
date: 2019-05-10 22:35:22
categories: 分布式系统
tags:
---

## 为什么要使用分布式锁

对共享变量或者数据（数据库数据）多线程操作的时候，为了避免在同一个时刻多线程操作导致数据的混乱，我们可以通过线程的互斥等来控制某个时刻只有一个线程执行，但这只是再单机应用下。

但是，随着业务的发展，架构随之发展，就需要做集群，负载均衡，这样子，同一个方法或者变量就会以多线程的方式且再不同的物理机也就是不同的jvm上同时执行，这显然是不对的，我们想要的是在某个时刻，该方法或者共享数据只在一个线程中执行，也就是只在某一台物理机子上执行，其余的排斥。这时，分布式锁就派上用场了，它就是做这么一件事的。

## 分布式锁应该具备哪些条件

在分析分布式锁的三种实现方式之前，先了解一下分布式锁应该具备哪些条件：

>1、在分布式系统环境下，一个方法在同一时间只能被一个机器的一个线程执行； 
2、高可用的获取锁与释放锁； 
3、高性能的获取锁与释放锁； 
4、具备可重入特性； 
5、具备锁失效机制，防止死锁； 
6、具备非阻塞锁特性，即没有获取到锁将直接返回获取锁失败

## 常用分布式锁实现方式

>基于数据库实现分布式锁； 
基于缓存（Redis等）实现分布式锁； 
基于Zookeeper实现分布式锁；

### 1.基于Redis缓存实现分布式锁

#### Spring Boot使用RedLock实现分布式锁




### 2.基于Zookeeper实现分布式锁


### 3.基于数据库实现分布式锁


https://www.cnblogs.com/seesun2012/p/9214653.html