---
layout: post
title:  "单例模式"
date:   2019-05-03 21:55:51 +0800
categories: Summary
tags: Java 设计模式
description: Java设计模式之单例模式
---

### 一、单例的四大原则: 

1. 构造私有。   
2. 以静态方法或者枚举返回实例。   
3. 确保实例只有一个，尤其是多线程环境。   
4. 确保反序列换时不会重新构建对象。  

### 二、常见的几张实现模式
**饿汉模式、懒汉模式、双重检查懒汉模式、静态内部类模式、枚举模式**

1.饿汉模式:过早创建对象占空间，但是线程安全
{% highlight ruby %}
//饿汉式有两种
public class SingletonTest01{

    private SingletonTest01(){

    }
    //类装载过程
    private final static SingletonTest01 instance = new SingletonTest01();

    static SingletonTest01 getInstance() {
        return instance;
    }
}

//饿汉式（静态代码块）
public class SingletonTest01 {

    private SingletonTest01(){}

    private static SingletonTest01 instance;
    //类加载过程
    static {
        instance = new SingletonTest01();
    }

    public static SingletonTest01 getInstance(){
        return instance;
    }
}

{% endhighlight %} 


<br/>

2.懒汉式: 只有在调用的时候创建对象，占用空间小，但加锁的粒度太大，效率低，较少使用
{% highlight ruby %}
//懒汉式（线程安全，同步方法，效率低）
public class SingletonTest02 {
    private static SingletonTest02 instance;

    private SingletonTest02(){}

    public static synchronized SingletonTest02 getInstance(){
        if(instance == null){
            instance = new SingletonTest02();
        }
        return instance;
    }
}
{% endhighlight %} 

3.双重检查懒汉式: 占用空间小，执行效率更高，常用。   
由于jvm存在乱序执行功能，有可能DCL失效问题，在JDK1.6及以后，只要定义为private volatile static SingleTon INSTANCE = null;就可解决DCL失效问题。volatile确保INSTANCE每次均在主内存中读取，这样虽然会牺牲一点效率，但也无伤大雅。
{% highlight ruby %}
//双重检查
public class SingletonTest03 {

    private SingletonTest03(){}

    private static volatile SingletonTest03 instance;

    public static SingletonTest03 getInstance(){
        if(instance == null){
            synchronized (SingletonTest03.class){
                if(instance == null){
                    instance = new SingletonTest03();
                }
            }
        }

        return instance;
    }
}
{% endhighlight %} 

4.内部静态类: 虚拟机会保证一个类的<clinit>()方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的<clinit>()方法，其他线程都需要阻塞等待，直到活动线程执行<clinit>()方法完毕。如果在一个类的<clinit>()方法中有耗时很长的操作，就可能造成多个进程阻塞(需要注意的是，其他线程虽然会被阻塞，但如果执行<clinit>()方法后，其他线程唤醒之后不会再次进入<clinit>()方法。    
同一个加载器下，一个类型只会初始化一次。)，在实际应用中，这种阻塞往往是很隐蔽的。   
缺点: 无法从外部传参
{% highlight ruby %}
//内部静态类（JVM帮助我们保证了线程的安全性，在类进行初始化时，别的线程是无法进入的）
public class SingletonTest04 {

    private SingletonTest04(){}

    public static SingletonTest04 instance;

    private static class SingletonInstance{
        private static final SingletonTest04 INSTANCE = new SingletonTest04();
    }

    public static SingletonTest04 getInstance(){
        return SingletonInstance.INSTANCE;
    }
}
{% endhighlight %} 
