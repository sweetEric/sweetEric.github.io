---
layout: post
title:  "Spring Security配置"
date:   2019-07-10 20:35:53 +0400
categories: SpringBoot
tags: summary
author: Eric
description: SpringBoot配置Spring Security进行认证授权
---  


认证
Spring Security所解决的问题就是安全访问控制，而安全访问控制功能其实就是对所有进入系统的请求进行拦截，校验每个请求是否能够访问它所期望的资源。根据前边知识的学习，可以通过 Filter或AQP等技术来实现，Spring Security对Web资源的保护是靠Fite实现的，所以从这个Fter来入手，逐步深入 Spring Security原理。
当初始化 Spring Security时，会创建一个名为 SpringSecurityFilterChain的servlet过滤器，类型为org.springframework.security.web.FilterChainProxy，它实现了javax.servlet.Filter，因此外部的请求会经过这个类

FilterChainProxy是一个代理，真正起作用的是FilterChainProxy中 SecurityFilterChain所包含的各个Filter同时这些Filter作为Bean被Spring管理，它们是Spring Security核心，各有各的职责，但他们并不直接处理用户的认证，也不直接处理用户的授权，而是把它们交给了认证管理器（AuthenticationManager）和决策管理器
（AccessDecisionMfanager）进行处理，下图是 FilterChainProxy相关类的UML图示


UserDetailsService 改写获取用户信息的组件
PasswordEncoder

授权
AccessDecisionManager
AbstractAccessDecisionManager
AffirmativeBased

Web授权(建议通过权限进行授权)
从上到下逐层拦截
{% highlight ruby %} 
 @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .antMatchers("/r/r1").hasAuthority("p1")
                .antMatchers("/r/r2").hasAuthority("p2")
                .antMatchers("/r/**").authenticated()
                .anyRequest().permitAll()
                .and()
                .formLogin()
                .loginPage("/pc_login")
                .loginProcessingUrl("/login")
                .successForwardUrl("/login-success")
                .and()
                .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED);
    }
{% endhighlight %}   


方法授权
@EnableGlobalMethodSecurity(securedEnabled=true) 开启方法授权
@EnableGlobalMethodSecurity(preAuthorizeEnabled=true)
@PreAuthorize("isAnonymous()")
@PostAuthorize
@Secured("IS_AUTHENTICATED_ANONYMOUSLY") 
IS_AUTHENTICATED_ANONYMOUSLY   ROLE_TELLER

@PrePost


HttpSecurity进行Spring Security的基础配置
授权的方式包括web授权和方法授权，web授权是通过ur拦截进行授权，方法授权是通过方法拦截进行授权。他们都会调用 access Decision Manager进行授权决策，若为web授权则拦截器为 FilterSecuritylnterceptor若为方法授权则拦截器为 MethodSecurityInterceptor。如果同时通过web授权和方法授权则先执行web授权，再执行方法授权，最后决策通过，则允许访问资源，否则将禁止访问。

会话控制
