---
layout: post
title:  "JVM 调优"
date:   2019-08-2 21:11:56 +0800
categories: JVM
tags: summary
author: Eric
description: 了解Jvm调优的内容, 认识调优的常见参数, 知道如何选择垃圾收集
---

### 一、JVM内存模型

Jvm调优主要是堆内存调优, 也叫jvm堆调优。    

![jvm内存模型]({{ "/assets/images/postImg/JVM-memery-module.png" | absolute_url }})   


每一块子内存区中都会存在有一部分的可变伸缩区, 其基本流程：
如果空间不足, 在可变的范围之内扩大内存空间, 当一段时间之后发现内存空间没有这么紧张的时候, 再将可变空间进行释放。   

![Jvm堆内存空间分配]({{ "/assets/images/postImg/Jvm-heap-deliver.png" | absolute_url }})

![Jvm空间调整参数]({{ "/assets/images/postImg/Jvm-space-adjust.png" | absolute_url }})
     
------  

### 二、几个调优的方向
1. 减少fullGC的次数
2. 减少fullGC的时间
3. 减少自动调整堆大小次数
     
------  

### 三、Jvm调优方法
1. 直接将gc.log放到一些jvm分析网站上分析
网址：https://www.gceasy.io/   

2. 通过命令 jmap -heap PID 查看运行内存使用情况

3. 可视化工具：jvisualvm (命令行执行此命令) 或 jconsole


     
------  

### 四、主要参数分析
**建议将gc日志放到专业的网站上分析, 对整个jvm的使用情况有更清晰的了解**

#### JVM memory size

| Young Generation | 246.67 mb | n/a       |
| Old Generation   | 493.33 mb | n/a       |
| Total            | 740 mb    | 510.46 mb |   


#### 性能参数   

| 吞吐量(Throughput) |  99.995% |
| 延迟(Latency)    |          |
| Avg Pause GC Time  | 13.9 ms  |
| Max Pause GC Time  | 89.6 ms  |    

------     

| 持续时间   | GC数量 | 百分比     |
|--------|------|---------|
| 0\-10  | 3    | 0\.21％  |
| 10至20  | 1328 | 94\.45％ |
| 20至30  | 46   | 3\.27％  |
| 30\-40 | 19   | 1\.35％  |
| 40\-50 | 5    | 0\.36％  |
| 60\-70 | 2    | 0\.14％  |
| 70\-80 | 1    | 0\.07％  |
| 80\-90 | 2    | 0\.14％  |
     
------  

### 五、Jvm优化
- 修改JVM内存配置, 尽量减少Jvm自动伸缩对空间, 
- 响应时间优先的应用    

    年老代尽可能设大, 直到接近系统的最低响应时间限制(根据实际情况选择)。在此种情况下, 年轻代收集发生的频率也是最小的。同时, 减少到达年老代的对象。年老代使用并发收集器, 所以其大小需要小心设置, 一般要考虑并发会话率和会话持续时间等一些参数。如果堆设置小了, 可以会造成内存碎片、高回收频率以及应用暂停而使用传统的标记清除方式；如果堆大了, 则需要较长的收集时间。   


{% highlight ruby %} 
    -Xmx760m  	                          //分配内存大小
    -Xms660m		      	              //一般和Xmx一样
    -XX:NewRatio=2		      	           //老生区和新生区新生区之比2:1
    -XX:SurvivorRatio=6		              //Eden区和suvice区之比为6:1
    -XX:CMSInitiatingOccupancyFraction=85  //当老生区达到85%时执行GC
    -XX:+UseConcMarkSweepGC               //使用并发收集器(ParNew+CMS)
    -XX:CMSFullGCsBeforeCompaction=5         //老生代执行7次GC时执行碎片整理
{% endhighlight %}     


- 吞吐量优先的应用     

    设置较大的年轻代。原因是, 这样可以尽可能回收掉大部分短期对象, 减少中期的对象, 而年老代存放长期存活对象。 垃圾收集可以并行进行, 一般适合 8CPU 以上的应用。

{% highlight ruby %} 
    -Xmx760m  	                          //分配内存大小  
    -Xms660m		      	               //一般和Xmx一样
    -XX:NewRatio=2		      	            //老生区和新生区新生区之比1:1
    -XX:SurvivorRatio=6		             //Eden区和suvice区之比为6:1
    -XX:+UseCMSCompactAtFullCollection：使用并发收集器时, 开启对年老代的压缩。
    -XX:+UseParallelGC                      //虚拟机在Server模式下的默认值, Parallel Scavenge+Serial Old
    <!-- -XX:ParallelGCThreads=2 -->
    -XX:MaxGCPauseMillis=100               //提高年轻代 GC 效率的配置。次收集器执行效率。 
    -XX:+UseAdaptiveSizePolicyUseAdaptiveSizePolicy 
    -XX:+UseCompressedClassPointers;
    -XX:+UseCompressedOops;
    -XX:+UseConcMarkSweepGC;
    -XX:-UseLargePagesIndividualAllocation;
{% endhighlight %} 

```
    测试下使用   
    -XX:+PrintGC    
    -XX:+Printetails   
    -XX:+PrintGCTimeStamps 
    -Xloggc:filename     
```


![JVM-tuning-strategy]({{ "/assets/images/postImg/JVM优化策略.png" | absolute_url }})   

    注：
    startup.bat启动：在apache-tomcat-8.5.43\bin\catalina.bat第一行增加 
    系统服务启动：在Tomcat的service.bat中185行中增加JVM参数,修改JvmMx变量, JvmMs变量set JAVA_OPTS=....
     
------  

### 六、GC处理器总结(查找)：  
- new: Serial收集器(复制算法), 单线程收集, 进行垃圾收集时, 必须暂停所有工作线程     

- new: Parallel Scavenge收集器(复制算法), 多线程收集, 其余的行为、特点和Serial收集器一样, 单核执行效率不如Serial, 在多核下执行才有优势, 比起关注用户线程停顿时间, 更关注系统的吞吐量    

- old: Parallel Old收集器(标记 - 整理算法), 多线程, 吞吐量优先    

- CMS收集器(标记 - 清除算法): 1.初始标记：stop - the - worl, 2.并发标记,并发追溯标记, 程序不会停顿, 3.并发预清理：查找执行并发标记阶段从年轻代晋升到老年代的对象, 3.重新标记：暂停虚拟机, 扫描CMS堆中的剩余对象, 4.并发清理：清理垃圾对象, 程序不会停顿,  5.并发重置：重置CMS收集器的数据结构。    

- G1收集器(复制 + 标记 - 整理算法): 1.并行和并发, 2.分代收集, 3.空间整合(解决内存碎片问题), 4.可预测的停顿, 可以非常精确地控制停顿      

| 参数     | 描述   |
| ----------------------- | ---------------------------------------------------- |
| -XX:+UseSerialGC,      | Client模式下的默认值, Serial + Serial Old    |
| -XX:+UseConcMarkSweepGC | ParNew+CMS                                           |
| -XX:+UseParallelGC      | Server模式下的默认值, Parallel Scavenge + Serial Old |
| -XX:+UseParallelOldGC   | Parallel Scavenge+Parallel Old                       |
| -XX:+UseG1GC            |  G1 Young Generation + G1 Old Generation             |
| -XX:-UseParallelOldGC   | ParallelScavenge + SerialOld                         |      

------  

| 垃圾回收器 | 新生代名     | 老年代名                              |
| ----------- | ---------------- | ----------------------------------------- |
| G1GC        | G1New            | G1Old                                     |
| Parallel GC | ParallelScavenge | ParallelOld(-UseParallelOld则是SerialOld) |
| CMS         | ParNew           | ConcurrentMarkSweep                       |
| Serial GC   | DefNew           | SerialOld                                 |
| Epsilon     | N/A              | N/A                                       |
| ZGC         | N/A              | Z                                         |
| Shenandoah  | N/A              | Shenandoah                                |   



### 引用:   


###### [https://www.cnblogs.com/leo-chen-2014/p/9739563.html][blog1-url]   
###### [https://www.cnblogs.com/kelthuzadx/p/10924117.html][blog2-url]

[blog1-url]: https://www.cnblogs.com/leo-chen-2014/p/9739563.html
[blog2-url]: https://www.cnblogs.com/kelthuzadx/p/10924117.html
