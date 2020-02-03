---
layout: post
title:  "物联网TIME_WAIT数量过多"
date:   2019-04-03 21:20:56 +0800
categories: Summary
tags: Server
author: Eric
description: 服务器启动时time_wait状态的连接过多
---


### 一、问题原因   
- 默认Windows Server 2008版本以后，动态端口的数量为16384个 (从49152起始，到65536结束)，如果服务器对外有大量连接，而根据TCP默认的Time Wait Delay时间为4分钟，这会导致大量连接在断开后处于Time_Wait状态，无法快速释放给其它连接使用，会导致端口耗尽。    

### 二、解决方法
1. 通过管理终端登录Windows系统的ECS实例中，在CMD界面执行如下命令，查看当前动态端口配置
netsh int ipv4 show dynamicport tcp
2. 执行如下命令，增大动态端口数量，无需重启即可生效
netsh int ipv4 set dynamicport tcp start=1025 num=60000
3. 如果以上配置不能完全解决，可以通过修改注册表降低Time Wait时间，最低为30秒。打开注册表，定位到HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters，新增键值TcpTimedWaitDelay，类型REG_DWORD , 设置为十进制30
