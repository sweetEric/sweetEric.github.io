---
layout: post
title:  "原型模式"
date:   2019-05-04 22:50:51 +0800
categories: Summary
tags: 设计模式 Java
author: Eric
description: Java设计模式之原型模式
---

### 一、定义   
这里套用菜鸟教程上的定义: 原型模式（Prototype Pattern）是用于创建重复的对象，同时又能保证性能。这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。当直接创建对象的代价比较大时，则采用这种模式。

<br/>

### 二.原型模式创建步骤
1. Prototype: 原型类, 声明一个克隆自己的接口 Clonable
2. ConcretePrototype: 具体的原型类, 实现一个克隆自己的操作
3. Client: 让一个原型对象克隆自己, 从而创建一个新的对象    

{% highlight ruby %}
//Sheep.java
public class Sheep implement Clonable{
@Override
protected Object clone()
{
	Sheep sheep = null;
	try{
		sheep = (Sheep)super.clone();
	}catch(Exception e){
		------
	}

	return super.clone();
}      

//Client.java
public class Client{
	public static void main(String[] args){
		Sheep sheep = new Sheep("",---);
		Sheep sheep2 = (Sheep)sheep.clone();
		Sheep sheep3 = (Sheep)sheep.clone();
	}
}    
{% endhighlight %} 
	
<br/>

### 三、原型模式分析    
```
1.在上述的clone方法中使用的是深拷贝还是浅拷贝
答: 浅拷贝, 当类内部属性有类对象时, clone方法会将他们的的引用复制一份。 

2.实现深拷贝的方式     
方法1：内部类继承Clonable接口。重写clone接口时, 先进行内部类clone    
方法2：通过对象的序列化实现（推荐）, 代码如下   
```   

{% highlight ruby %}
//Sheep.java
public Object deepClone(){
	ByteArrayOutputStream bos = null;
	ObjectOutputStream oos = null;
	ByteArrayInputStream bis = null;
	ObjectInputStream ois = null;
	try{
		bos = new ByteArrayOutputStream();
		oos = new ObjectOutputStream(bos);
		oos.writeObject(this);
		bis = new ByteArrayInputStream(bos.toByteArray());
		ois = new ObjectInputStream(bis);
		Sheep copyObj = (Sheep)ois.readObject();
		return copyObj;
	}catch(Exception e){
		-----
		return null;
	}finally{
		try{
			bos.close();
			oos.close();
			bis.close();
			ois.close();
		}catch(Exception e){
			-----
		}
	}
}    

//Client.java   
public class Client{
	public static void main(String[] args){
		Sheep sheep = new Sheep("",---);
		Sheep sheep2 = (Sheep)sheep.deepClone();
		
	}
}     
{% endhighlight %} 


---
## 四、总结
1. 创建新的对象比较复杂时, 可以利用原型模式简化对象的创建过程, 同时也能够提供高效率。     
2. 通过原型模式不用重新初始化对象, 而是动态地获得对象运行时的状态。    
3. 在实现深克隆的时候可能需要比较复杂的代码, 如果使用一个类配备一个的方式克隆, 对已有的类改造时, 要修改源代码, 违背OCP原则。   

---
	
		

