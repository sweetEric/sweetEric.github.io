---
layout: post
title:  "Redis 的几种常见使用场景"
date:   2019-04-05 28:10:51 +0800
categories: Summary
tags: SpringBoot Redis
description: 具体分析redis的集几种数据类型的不同和相关使用场景
---

>Springboot2.0

### 一、Reids基础介绍
>redis是一个开源的使用C语言编写的一个kv存储系统，是一个速度非常快的非关系远程内存数据库。它支持包括String、List、Set、Zset、hash五种数据结构。除此之外，通过复制、持久化和客户端分片等特性，用户可以很方便地将redis扩展成一个能够包含数百GB数据和每秒处理上百万次的请求的系统。目前支持多种语言的api，方便用户使用。   
redis同时也内置了事务、LUA脚本、复制等功能，提供两种持久化选项，一种是每隔一段时间将数据导入到磁盘(快照模式)，另一种是追加命令到日志中(AOF模式)。如果只是作为高效的内存数据库使用也可以关闭持久化功能。通过哨兵(sentinel)和自动分区(Cuuster)的方式可以提高redis服务器的高可用性。
与关系型数据库相比，redis的命令请求不需要经过查询分析器或查询优化器进行处理，也避免了更新数据时引起的随机读\写，这些慢操作。它直接读写内存中的数据，并且数据是按照一定的数据结构存储的。所以它的速度非常快。   


### 二、Redis5种数据类型使用场景梳理
1. String字符串类型
    >1.Redis支持的字符串类型不是定长分配的字符串，是动态变长字符串，修改字符串在没有增加特别多内容的情况下不需要重新分配内存空间，内部结构实现上有点类似于java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配。常用于以json形式缓存用户信息。   
    >2.计数功能:redis是单线程运行,就算client是多线程调用那么也是正确自增,实时计数统计场景，也用在过库存场景、限流计数场景等并发量没那么高的系统都是适用的。

2. List列表类型
redis的列表的数据结构和Java中的LinkedList比较类似，所以List类型的前后插入和删除速度是非常快的，但是随机定位速度非常慢，时间复杂度是O（n）需要对列表进行遍历。
>1.list列表结构常用来做异步队列使用      
2.list可用于秒杀抢购场景

3. Hash数据类型
hash字典类型也是比较适合保存结构体信息的，不同于字符串一次序列化整个对象，hash可以对用户结构中的每个字段单独存储。这样当我们需要获取结构体信息时可以进行部分获取，而不用序列化所有字段，而将整个字符串保存的结构体信息只能一次性全部读取。

4. Set集合类型
redis的set相当于java中的HashSet，内部的健值是无序唯一的，相当于一个hashmap，但是value都是null。使用场景也是比较单一的，就是用在一些去重的场景里，例如每个用户只能参与一次活动、一个用户只能中奖一次等去重场景

5. Sorted Set(zset)
当你需要一个有序的并且不重复的集合列表，那么可以选择sorted set数据结构，比如twitter 的public timeline可以以发表时间作为score来存储，这样获取时就是自动按时间排好序的。



### 资料  
1. expand: 首先创建一个比现有哈希表更大的新哈希表。
2. rehash: 然后将旧哈希表的所有元素都迁移到新哈希表去。
3. redis的hash使用渐进式rehash。

#### 引用
[塞冷鸿飞急][blog0-url]


[blog0-url]: https://blog.csdn.net/weixin_44098139/article/details/88672943