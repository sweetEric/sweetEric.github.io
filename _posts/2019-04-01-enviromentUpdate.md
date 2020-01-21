---
layout: post
title:  "后台项目运行环境升级"
date:   2019-04-01 21:10:51 +0800
categories: Windows
tags: pratice
author: Eric
description: 由于目前的项目搭建的平台太久, 需要升级
---

### 一、运行环境基础配置
1. 服务器环境
>2vCPU 4 GiB (I/O优化)   
>ecs.n4.large   5Mbps (峰值)    
>Intel(R) Xeon(R) CPU E5-2682 v4 @ 2.5GHz     
      
2. 运行环境
>jre: 1.6    
>jvm：默认参数    
>tomcat: 7    


### 二、影响项目运行效率的因子    
1. 服务器性能
2. 项目架子旧
3. Jvm配置不合理

### 三、升级流程   
- 使用JDK8：环境变量-系统变量-添加JAVA_HOME变量（参数: D:\Java\jdk1.8.0_221-在Path变量中添加（%JAVA_HOME%\jre\bin / %JAVA_HOME%\bin） -cmd输入java -v确认
- Tomcat8： 直接官网下载
- 更改tomcat监控用户： 在apache-tomcat-8.5.43\conf\tomcat-users.xml中增加    

{% highlight ruby %} 
    <tomcat-users> </tomcat-users>
    <role rolename="manager-gui"/>
    <user username="admin" password="admin" roles="manager-gui"/>
{% endhighlight %}   

<br/>

- 配置Tomcat服务   

    1.修改startup.bat文件    
 
    在第一行前面加入下面的语句:SET JAVA_HOME=%JDK路径% \n SET CATALINA_HOME=%自己的Tomcat路径%    

    2.修改 shutdown.bat文件   

    在第一行前面加入下面的语句:SET JAVA_HOME=%自己JDK路径% \n SET CATALINA_HOME=%自己的Tomcat路径%   

    3.修改service.bat文件   

    SET CATALINA_HOME=%自己的Tomcat路径%   

    SET SERVICE_NAME=tomcatServer (服务的名字,在命令行中通过该名字进行服务的控制(启动/关闭))    

    SET PR_DISPLAYNAME=tomcatServer (服务的显示名称,即在服务管理器中显示的名称.)   


    4.安装卸载Tomcat服务
    service install & service uninstall