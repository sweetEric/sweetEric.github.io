---
layout: post
title:  "Java 中四种创建对象的方法"
date:   2019-03-30 23:02:51 +0800
categories: Summary
tags: Java
author: Eric
description: Java 中四种创建对象的方法
---

在Java 中我们有4种常用的方式创建对象
1. 通过 new 创建
2. 通过反射创建
3. 通过cloneable接口创建
4. 通过反序列化获取对象

com.lwx.fourInstance.InstanceDemo.java
{% highlight ruby %} 
public interface InstanceDemo {

    String toString();
    void print(String args);
}

{% endhighlight %}  

<br/>

com.lwx.fourInstance.InstanceDemoImpl.java
{% highlight ruby %} 
package com.lwx.fourInstance;
import java.io.Serializable;

//Cloneable只有使用第三种方法才用到，Serializable只有使用第四种才用到
public class InstanceDemoImpl implements InstanceDemo, Cloneable, Serializable {

    @Override
    public void print(String args) {
        System.out.println("out: + " + args + "\n");
    }

    @Override
    protected InstanceDemoImpl clone() throws CloneNotSupportedException {
        // TODO Auto-generated method stub
        return (InstanceDemoImpl) super.clone();

    }
}
{% endhighlight %}  

<br/>

com.lwx.fourInstance.Main.java
{% highlight ruby %} 
package com.lwx.fourInstance;
import java.io.*;

public class Main {

    public static void main(String[] args){
	// write your code here
        //第一种
        new InstanceDemoImpl().print("LWX---new 创建");

        //第二种
        try{
            Class clazz = Class.forName("com.lwx.fourInstance.InstanceDemoImpl");
            InstanceDemoImpl demo = (InstanceDemoImpl) clazz.newInstance();
            demo.print("LWX---反射创建");
        }catch (ClassNotFoundException | IllegalAccessException | InstantiationException e){
            e.printStackTrace();
        }

        //第三种实现Cloneable接口
        InstanceDemoImpl instanceDemo = new InstanceDemoImpl();
        try{
            instanceDemo.clone().print("LWX---Clone接口创建");

        }catch (CloneNotSupportedException e)
        {
            e.printStackTrace();
        }


        //第四种序列化机制实现Serializable接口
        InstanceDemoImpl demo = new InstanceDemoImpl();
        File f = new File("Instance.obj");
        try{

            FileOutputStream fos = new FileOutputStream(f);
            ObjectOutputStream  oos = new ObjectOutputStream(fos);

            FileInputStream fis = new FileInputStream(f);
            ObjectInputStream ois = new ObjectInputStream(fis);

            oos.writeObject(demo);
            InstanceDemoImpl instanceDemo1 = (InstanceDemoImpl) ois.readObject();
            instanceDemo1.print("LWX---反序列化创建");

        }catch (IOException | ClassNotFoundException e){
            e.printStackTrace();
        }

    }
}
{% endhighlight %}    

```
运行输出结果:   
out: + LWX---new 创建

out: + LWX---反射创建

out: + LWX---Clone接口创建

out: + LWX---反序列化创建    
```