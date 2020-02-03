---
layout: post
title:  "TCP/IP协议详解"
date:   2019-03-10 20:55:53 +0400
categories: TCP
tags: Summary
author: Eric
description: TCP/IP协议归纳，总结
---   


### 一、TCP/IP协议的含义
　　从字面意义上讲，有人可能会认为 TCP/IP 是指 TCP 和 IP 两种协议。实际生活当中有时也确实就是指这两种协议。然而在很多情况下，它只是利用 IP 进行通信时所必须用到的协议群的统称。具体来说，IP 或 ICMP、TCP 或 UDP、TELNET 或 FTP、以及 HTTP 等都属于 TCP/IP 协议。他们与 TCP 或 IP 的关系紧密，是互联网必不可少的组成部分。TCP/IP 一词泛指这些协议，因此，有时也称 TCP/IP 为网际协议群。   
互联网进行通信时，需要相应的网络协议，TCP/IP 原本就是为使用互联网而开发制定的协议族。因此，互联网的协议就是 TCP/IP，TCP/IP 就是互联网的协议。
 
 -------

### 二、计算机网络结构分层   

>　　OSI七层模型分别为物理层、链路层、网络层、传输层、会话层、表示层、应用层，其中传输层中的TCP，UDP是最基本的协议族，应用层中的HTTP、FTP、DNS等高级协议族都以TCP或UDP为基础的。   

![计算机网络结构分层]({{ "/assets/images/postImg/network-7layer.png" | absolute_url }})   

-------

### 三、TCP三次握手，四次挥手
两端在握手传输时，会自动置位自身状态(如所处阶段: SYN__SENT、ESTABLISHED、SYN_RCVD、ESTABLISHED; 包序号:XXX)。   

**三次握手**
 1. 客户端发包SYN=1、seq=J,变为SYN_SENT。
 2. 服务端变为SYN_RCVD, 发包ACK=1、ack=J、SYN=1、seq=K。
 3. 客户端变为ESTABLISHED, 发包ACK=1，ack=K+1。服务端收到变为ESTABLISHED。    

 ![TCP三次握手]({{ "/assets/images/postImg/tcp-3handup.png" | absolute_url}})   

<br/>   

 两端在挥手过程时，会自动置位自身状态(如所处阶段: FIN_WAIT_1、CLOSE_WAIT、FIN_WAIT_2、LAST_ACK、TIME_WAIT、CLOSED; 包序号:XXX)。    

**四次挥手**
 1. 客户端发包FIN=1、seq=X, 变为FIN_WAIT_1,不能发。
 2. 服务端变为CLOSE_WAIT，发包ACK=1、ack=X+1,
 3. 服务端客户端发包FIN=1、seq=Y, 变为LAST_ACK,不能发。
 4. 客户端变为TIME_WAIT，发包ACK=1、ack=Y+1,等待2MSL CLOSED，服务端接收到CLOSED。   

![TCP四次挥手]({{ "/assets/images/postImg/tcp-4bye.png" | absolute_url }}) 

-------

### 四、TCP的可靠性
- 在 TCP 中，当发送端的数据到达接收主机时，接收端主机会返回一个已收到消息的通知。这个消息叫做确认应答（ACK）。当发送端将数据发出之后会等待对端的确认应答。如果有确认应答，说明数据已经成功到达对端。反之，则数据丢失的可能性很大。
- 在一定时间内没有等待到确认应答，发送端就可以认为数据已经丢失，并进行重发。由此，即使产生了丢包，仍然能够保证数据能够到达对端，实现可靠传输。
- TCP的有状态性和重发机制，握手挥手机制保证了TCP的可靠性。
