---
title: Spring IOC级联容器原理
date: 2019-05-12 20:12:00
tags: [spring]
categories: spring
---

## spring mvc框架中的两级容器
   - spring mvc在开发中经常使用，一般我们在web.xml配置时候，都需要配置ContextLoaderListener和一个DispatcherServlet，那么这2个都起到什么所用呢？
   - 其实就是配置了2个spring ioc容器，并且DispatcherServlet创建IOC容器的父容器就是ContextLoaderListener创建的IOC容器
   - 一般配置如下：
   ```
   配置 ContextLoaderListener 
   
      <listener>
          <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      </listener>
      
      <context-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>WEB-INF/applicationContext.xml</param-value>
      </context-param>
      
   配置 DispatcherServlet
   
      <servlet>
          <servlet-name>springmvc</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <load-on-startup>1</load-on-startup>
      </servlet>
   ```
   - ContextLoaderListener 会创建一个由应用程序上下文 XMLWebApplicationContext 来管理的 IOC 容器，IOC 里面的 bean 是通过 <context-param> 配置的 contextConfigLocation 参数对应的 WEB-INF/applicationContext.xml 来注入的
   ```
      <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
        <init-param>  
            <param-name>contextConfigLocation</param-name>  
            <param-value>classpath:springmvc-servlet.xml</param-value>  
        </init-param>
      </servlet>
   ```
   - 以上配置完成后就会形成两级级联IOC容器了
     - spring sub context ---------> spring root context  
    
## 总结
   - 1 子容器无法访问子容器的bean，因为相互隔离
   - 2 子容器可以访问父容器的bean,因为子容器也是通过父容器创建的
   - 3 父容器无法访问子容器中的bean,不支持自顶向下访问
   
