---
layout: post
title:  "SpringBoot 自动配置"
date:   2019-04-01 21:10:51 +0800
categories: Pratice
tags: SpringBoot
author: Eric
description: SpringBoot 的自动配置流程，常用的注解使用
---

### 一、 自动配置流程
- 1.@SpringBootApplication
SpringBoot 的自动配置从 **@SpringBootApplication** 该注解开始，该注解里面包含了一个自动配置注解注解 **@EnableAutoConfiguration**，如下。   

{% highlight ruby %}
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
{%  endhighlight %}

- 2.自动配置包
在 @AutoConfigurationPackage 中 Registrar.class 里有 registerBeanDefinitions 方法里面定义了自动扫描的包，也就是默认Application类中的当前目录下的所有包

{% highlight ruby %}
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}

-------

@Import({Registrar.class})
public @interface AutoConfigurationPackage {
}

-------

static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    Registrar() {
    }

    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        AutoConfigurationPackages.register(registry, (new AutoConfigurationPackages.PackageImport(metadata)).getPackageName());
    }

    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new AutoConfigurationPackages.PackageImport(metadata));
    }
}

{%  endhighlight %}

- 3.使用默认主配置类 
使用默认的配置，这里会给spring容器导入非常多的自动配置类
{% highlight ruby %}
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})

-------

protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    } else {
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        configurations = this.removeDuplicates(configurations);
        Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
        this.checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        configurations = this.filter(configurations, autoConfigurationMetadata);
        this.fireAutoConfigurationImportEvents(configurations, exclusions);
        return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
    }
}
{%  endhighlight %}

![自动导入的包列表]({{ "/assets/images/postImg/SpringBoot-AutoDependency.png" | absolute_url }})

### 二、 配置相关注解   
- @ConfigureationProperties(prefix = "xxx")   
    该注解和@Bean 或者 @Component 等只要能生成spring bean 的注解 结合起来使用,这样的话,当其他类注入该类时,就会触发该类的加载过程，注入在application.properties中以server开头的属性。   
- @Value()   
Spring容器从application.properties文件中读取指定的值。
- @ImportResource(locations = {classpath:xxx.xml})   
    导入Spring的**配置文件**，让配置文件里面的内容生效。   
- @Import   
    功能类似XML配置的，用来导入**配置类**   
- @PropertyResource("classpath:xxx.properties/yml")   
    是加载指定的**属性文件**。   



