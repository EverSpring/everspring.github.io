---
title: Arthas查看SpringBoot配置及ognl ClassNotFoundException处理
date: 2022-03-27 22:34:05
author: EverSpring
top: true
toc: false
mathjax: false
categories: JAVA
tags:
  - Arthas
  - Java
---
能讲到这篇文章的同学已经知道Arthas是什么了，这里就不多余赘述，本文介绍一下通过Arthas查看SpringBoot工程的配置及曾遇到过的问题。
本文相关知识：`sc`，`ognl`，`spring配置保存的地方`
> 注意：ognl是3.5.0以后出现的
1. arthas有可以查看JVM环境变量的`sysenv`，也有可以查看和修改JVM的系统属性的`sysprop`，但SpringBoot配置文件的内容不是在环境变量和系统属性中的，而是在`ConfigurableApplicationContext`的`environment`。`ConfigurableApplicationContext`继承自`ApplicationContext`，在我们的工程中经常要用到ApplicationContext，可以从ApplicationContext入手拿到environment。
2. 通过ognl表达式获取，ognl在java中运用广泛，比如Spring Cache的注解、Spring @Value注解都有用到。假如工程下有一个com.xxx.spring.util.SpringUtils（见文末）可以获取到ApplicationContext，想要获取到application.yml中在server.port的值，可以采用如下方法
```
ognl '#context=@com.xxx.util.SpringUtils@applicationContext,#context.getEnvironment().getProperty("server.port")'
```
> #context --代表的变量
> @com.xxx.util.SpringUtils --指定的类
> @applicationContext --类的属性
> #context.getEnvironment().getProperty("server.port") --通过context获取到environment属性，然后再通过environment获取到property的值
3. 有些情况下通过ognl获取applicationContext的时候会出现`ClassNotFoundException`，详情如下：
> Failed to execute ognl, exception message: ognl.OgnlException: Could not get static field applicationContext from class com.xxx.util.SpringUtils [java.lang.ClassNotFoundException: Unable to resolve class: com.xxx.util.SpringUtils], please check $HOME/logs/arthas/arthas.log for more details.

出现此类问题是由于classloader不是默认加载器导致，见官网issue：https://github.com/alibaba/arthas/issues/71
**解决方法：** 
* 1.通过sc -d 查询出当前类的classloader
```
sc -d com.xxx.util.SpringUtils
```
<img src="Arthas查看SpringBoot配置及ognl ClassNotFoundException处理/image-20220327234537394.png" alt="图1" style="zoom: 50%;" />

获取到classLoaderHash值
* 2.使用指定的classloader。ognl -c classloader的hash值
```
ognl '#context=@com.jinju.juke.common.utils.spring.SpringUtils@applicationContext,#context.getEnvironment().getProperty("dev.name")' -c 20ad941
```
---
SpringUtils的代码
```java
@Component
public final class SpringUtils implements BeanFactoryPostProcessor, ApplicationContextAware
{
    /** Spring应用上下文环境 */
    private static ConfigurableListableBeanFactory beanFactory;

    private static ApplicationContext applicationContext;

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException
    {
        SpringUtils.beanFactory = beanFactory;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException
    {
        SpringUtils.applicationContext = applicationContext;
    }

    /**
     * 获取对象
     *
     * @param name
     * @return Object 一个以所给名字注册的bean的实例
     * @throws org.springframework.beans.BeansException
     *
     */
    @SuppressWarnings("unchecked")
    public static <T> T getBean(String name) throws BeansException
    {
        return (T) beanFactory.getBean(name);
    }
}
```