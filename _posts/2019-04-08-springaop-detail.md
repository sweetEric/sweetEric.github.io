---
layout: post
title:  "Spring Aop 详解"
date:   2019-04-08 20:10:51 +0800
categories: Summary
tags: SpringBoot Java
author: Eric
description: Spring aop的使用，以及源码分析
---

>Springboot2.0

### 一、Aop是什么   

与OOP对比，面向切面，传统的OOP开发中的代码逻辑是自上而下的，而这些过程会产生一些横切性问题，这些横切性的问题和我们的主业务逻辑关系不大，这些横切性问题不会影响到主逻辑实现的，但是会散落到代码的各个部分，难以维护。AOP是处理些横切性问题，AOP的编程思想就是把这些问题和主业务逻辑分开，达到主业务逻辑解耦的目的，使代码的重用性和开发效率更高。  
使用场景
1. 日志记录
2. 权限验证
3. 效率检查
4. 事务管理    

<br/>

### 二、AspectJ 和 spring aop 的区别
AspectJ 为静态代理, 而spring aop包括JDK代理和cglib代理, 都是动态代理。

<br/>

### 三、Spring aop使用流程
如果想详细了解 Spring aop 的细节可参考官方文档    
 https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop
1. 创建代理
2. 创建切点(point cut)
3. 创建通知(advice)execution()

现在我们尝试创建动态代理，进行面向切面编程。    
可参考 https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#aop-ataspectj

{% highlight ruby %} 
//创建接口 A
public interface A {
    void print();
}

//创建组件AA
@Component("AA")
public class AA implements A{
    @Override
    public void print(){
        System.out.println(this.getClass());
    }

    @Override
    public String toString() {
        return "AA++";
    }
}

//执行的主逻辑
@RunWith(SpringRunner.class)
@SpringBootTest
public class AspectjTest {

    @Test
    public void Aspectj(){
        ConfigurableApplicationContext context = SpringApplication.run(HpmontApplication.class);

        System.out.println("\n");
        A a = (A) context.getBean("AA");
        a.print();
    }

}
{% endhighlight %}   

#### 现在我们对这个组件进行切面编程，也就是增强
{% highlight ruby %} 
//启用代理
@Configuration
@EnableAspectJAutoProxy
public class AppConfig {

}

//定义代理类
@Component
@Aspect
public class TestAspectJ {

    @Pointcut("execution(* com.hpmont.springiot.aspectj..*.*(..))")
    public void pointCut(){

    }

    @Before("pointCut()")
    public void advice()
    {
        System.out.println("aop before--logging");
    }

    @After("pointCut()")
    public void advice1()
    {
        System.out.println("aop after--logging");
    }

}

启用代理前
class com.hpmont.springiot.aspectj.AA

启用代理后
aop before--logging
class com.hpmont.springiot.aspectj.AA
aop after--logging

{% endhighlight %}    

>创建切点需要遵循相关语法    
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)     
这里问号表示当前项可以有也可以没有，其中各项的语义如下:    
modifiers- pattern:方法的可见性，如 public, protected    
ret-type- pattern:方法的返回值类型，如int,void等declaring-type- pattern:方法所在类的全路径名，如com, spring. Aspect;    
name- pattern:方法名类型，如 buisinessservice()   
param- pattern:方法的参数类型，如java.lang. string;    
throws- pattern:方法抛出的异常类型，如java.Lang.Exception     
example:   
pointcut("execution(* com.chenss.dao.*.*(..))")//配com. chess.dao包下的任意接囗和类的任意方法    
pointcut("execution( pubLic * com.chenss.dao.*.*(..))")//匹配com, chess.dao包下的任意接口和类的 public方法     
@Pointcut("execution(pubLic* com.chenss.dao.*.*())")//gUEL com, chess.dao包下的任意接口和类的 pubLic无方法参数的方法     
@Pointcut("execution(* com.chens dao.*.*(java.lang.String, ..))")//20配com, chess.dao包下的任意接囗和类的第一个参数为 string类型的方法    

<br/>

### 三、spring aop 源码分析   
 spring中的aop和AspectJ实现方式是不一样的aop使用的动态代理，AspectJ使用的是静态代理，我主要分析的是spring aop的JDK动态代理。通过面向切面编程，我们不会影响原OOP流程, 只是在原来的流程上切面注入新的功能，有以下两种方式    
1. 通过继承OOP流程中的某个类，对某个节点进行增强    
2. new 一个代理类把原有方法复制一份，并对其增强。   
![spring-oop-aop]({{ "/assets/images/postImg/spring-oop-aop.png" | absolute_url }})     

我们需要从源码中找出动态代理类是如何替换原来的类的并怎样进行增强。在spring bean中，beanfactory存储了所有我们自定义或者系统默认的bean，其中大部分在singletonObjects中维护，我们从singletonObjects 开始调试并找出是怎么维护bean map的。

{% highlight ruby %} 
//DefaultSingletonBeanRegistry.java
protected void addSingleton(String beanName, Object singletonObject) {
  synchronized (this.singletonObjects) {
    //在这里加断点
    this.singletonObjects.put(beanName, singletonObject);
    this.singletonFactories.remove(beanName);
    this.earlySingletonObjects.remove(beanName);
    this.registeredSingletons.add(beanName);
  }
}
{% endhighlight %}    

![debug-aop]({{ "/assets/images/postImg/debug-aop.png" | absolute_url }}) 

{% highlight ruby %} 
//找到 doGetBean 
// Create bean instance.
if (mbd.isSingleton()) {
  sharedInstance = getSingleton(beanName, () -> {
    try {

      //这里是创建bean
      return createBean(beanName, mbd, args);
    }
    catch (BeansException ex) {
      // Explicitly remove instance from singleton cache: It might have been put there
      // eagerly by the creation process, to allow for circular reference resolution.
      // Also remove any beans that received a temporary reference to the bean.
      destroySingleton(beanName);
      throw ex;
    }
  });
  bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}   

//往下走看到
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {
  ...
  try {

    //注意这里
    Object beanInstance = doCreateBean(beanName, mbdToUse, args);
    if (logger.isTraceEnabled()) {
      logger.trace("Finished creating instance of bean '" + beanName + "'");
    }
    return beanInstance;
  }
  ...
}


//往下走看到
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {
  ...
  try {
    populateBean(beanName, mbd, instanceWrapper);
    //注意这里
    exposedObject = initializeBean(beanName, exposedObject, mbd);
  }
  ...
}

//往下走看到
protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
  if (System.getSecurityManager() != null) {
    AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
      invokeAwareMethods(beanName, bean);
      return null;
    }, getAccessControlContext());
  }
  else {
    invokeAwareMethods(beanName, bean);
  }

  //这里把bean给了wrappedBean
  Object wrappedBean = bean;
  if (mbd == null || !mbd.isSynthetic()) {
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
  }

  try {
    invokeInitMethods(beanName, wrappedBean, mbd);
  }
  catch (Throwable ex) {
    throw new BeanCreationException(
        (mbd != null ? mbd.getResourceDescription() : null),
        beanName, "Invocation of init method failed", ex);
  }
  if (mbd == null || !mbd.isSynthetic()) {

    //在这里之后bean的名称发生了改变，方法也修改了
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
  }

  return wrappedBean;
}

//往下走看到
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {

  Object result = existingBean;
  for (BeanPostProcessor processor : getBeanPostProcessors()) {
    Object current = processor.postProcessAfterInitialization(result, beanName);
    if (current == null) {
      return result;
    }
    result = current;
  }
  return result;
}

//往下走看到
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {

  Object result = existingBean;
  for (BeanPostProcessor processor : getBeanPostProcessors()) {
    proxyTargetClass=true; optimize=false; opaque=false; exposeProxy=false; frozen=false
    //会在声明的代理中找是否有需要代理的bean，有就会创建代理进行替换
    Object current = processor.postProcessAfterInitialization(result, beanName);
    if (current == null) {
      return result;
    }
    result = current;
  }
  return result;
}
{% endhighlight %}   

<br/>

**总结一下: 由上面过程我们知道，spring是在需要进行切面编程的bean创建的时候创建的动态代理，并用动态代理替换掉原来的bean, 从而实现spring aop的。**

<!-- 参考
[Java架构师课程——spring AOP核心底层原理教程全集][aop]
[aop]: https://www.bilibili.com/video/av66845675 -->

<!-- aop和ioc关系    
其实这个题目本身意义不大，IOC和AOP都是编程目标和g没有关系，就算没有spring也能实现AOP，比如大名鼎鼎的 Aspect技术也能实现Aop。如果硬是要说关系应该说 spring的IoC和springAop有什么关系,无非 springAop当中的对象必须在Ioc容器当中。比如Aspect就可以脫离OC单独使用，但是 springAop就不能脱离Ioc。 -->