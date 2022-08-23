---
title: MDC在多线程中的使用
date: 2022-08-23 12:57:32
author: EverSpring
top: true
toc: false
mathjax: false
categories: JAVA
tags:
  - MDC
  - 多线程日志
---
# 一、了解MDC
MDC（Mapped Diagnostic Context，映射调试上下文）是 slf4j提供的一种轻量级的日志跟踪工具。Log4j、Logback或者Log4j2等日志中最常见区分同一个请求的方式是通过线程名，而如果
请求量大，线程名在相近的时间内会有很多重复的而无法分辨，因此引出了trace-id，即在接收到的时候生成唯一的请求id，在整个执行链路中带上此唯一id.
MDC.java本身不提供传递traceId的能力，真正提供能力的是`MDCAdapter`接口的实现。比如Log4j的是`Log4jMDCAdapter`，Logback的是`LogbackMDCAdapter`。
# 二、MDC普通使用
1. MDC.put("trace-id", traceId); // 添加traceId
2. MDC.remove("trace-id"); // 移除traceId
3. 在日志配置文件中`<pattern>`节点中以`%X{trace-id}`取出，比如：`<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level [uniqid-%X{trace-id}] %logger{50}-%line - %m%n</pattern>`
以上方式，只能在做put操作的当前线程中获取到值。那是因为MDC的实现原理主要就是ThreadLocal，ThreadLocal只对当前线程有效。

# 三、多线程中的MDC
要破除ThreadLocal只对当前线程有线的方法有两种：  
一种是JDK自带的、ThreadLocal的扩展类`InheritableThreadLocal`，子线程会拷贝父线程中的变量值
一种是引入alibaba`transmittable-thread-local`包的`TransmittableThreadLocal`实现  
> 据我所知，只有子线程获取父线程，不能父线程获取子线程的变量，如果有知道父获取子的方法，麻烦说一下，谢谢

Sl4j本身的提供的`BasicMDCAdapter`就是基于`InheritableThreadLocal`实现了线程间传递，但log4j、logback这两个实际的日志实现没有提供，log4j2提供了但默认关闭。
1. log4j2主要是根据`isThreadContextMapInheritable`变量控制的，有两种方法：
   * log4j2.component.properties文件中配置
   * 类型模块中定义,```System.setProperty("log4j2.isThreadContextMapInherimeble", "true");```
2. lob4j和logback需要自己手动实现，主要有两种方法，一是`手动在线程中的处理`，一种是重写`LogbackMDCAdapter`。本文以在线程池中创建为例
```java
import org.slf4j.MDC;
import org.springframework.util.CollectionUtils;

import java.util.Map;
import java.util.concurrent.Callable;

/**
 * @desc: 定义MDC工具类，支持Runnable和Callable两种，目的就是为了把父线程的traceId设置给子线程
 */
public class MdcUtil {

    public static <T> Callable<T> wrap(final Callable<T> callable, final Map<String, String> context) {
        return () -> {
            if (CollectionUtils.isEmpty(context)) {
                MDC.clear();
            } else {
                MDC.setContextMap(context);
            }
            try {
                return callable.call();
            } finally {
                // 清除子线程的，避免内存溢出，就和ThreadLocal.remove()一个原因
                MDC.clear();
            }
        };
    }

    public static Runnable wrap(final Runnable runnable, final Map<String, String> context) {
        return () -> {
            if (CollectionUtils.isEmpty(context)) {
                MDC.clear();
            } else {
                MDC.setContextMap(context);
            }
            try {
                runnable.run();
            } finally {
                MDC.clear();
            }
        };
    }
}
```
```java
import java.util.concurrent.Callable;
import java.util.concurrent.Future;

/**
 * @desc: 把当前的traceId透传到子线程特意加的实现。重点就是 MDC.getCopyOfContextMap()，此方法获取当前线程（父线程）的traceId
 */
public class ThreadPoolMdcExecutor extends ThreadPoolTaskExecutor {
    @Override
    public void execute(Runnable task) {
        super.execute(MdcUtil.wrap(task, MDC.getCopyOfContextMap()));
    }

    @Override
    public Future<?> submit(Runnable task) {
        return super.submit(MdcUtil.wrap(task, MDC.getCopyOfContextMap()));
    }

    @Override
    public <T> Future<T> submit(Callable<T> task) {
        return super.submit(MdcUtil.wrap(task, MDC.getCopyOfContextMap()));
    }
}
```

```java
import cn.hutool.core.util.ReflectUtil;
import com.ccblife.hiim.util.ThreadUtils;
import org.apache.commons.lang3.concurrent.BasicThreadFactory;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import javax.annotation.Resource;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.ScheduledThreadPoolExecutor;

/**
 * 线程池配置。重点就是把ThreadPoolTaskExecutor换成ThreadPoolMdcExecutor
 **/
@Configuration
public class ThreadPoolConfig {

    @Resource
    private ThreadPoolProperties threadPoolProperties;

    /**
     * 业务用到的线程池
     * @return
     */
    @Bean(name = "threadPoolTaskExecutor")
    public ThreadPoolTaskExecutor threadPoolTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolMdcExecutor();
        executor.setMaxPoolSize(threadPoolProperties.getMaxPoolSize());
        executor.setCorePoolSize(threadPoolProperties.getCorePoolSize());
        executor.setQueueCapacity(threadPoolProperties.getQueueCapacity());
        executor.setKeepAliveSeconds(threadPoolProperties.getKeepAliveSeconds());
        RejectedExecutionHandler handler = ReflectUtil.newInstance(threadPoolProperties.getRejectedExecutionHandler().getClazz());
        executor.setRejectedExecutionHandler(handler);
        return executor;
    }
}
```

推荐文章：
1. https://www.cnblogs.com/yangyongjie/p/12523567.html
2. https://blog.csdn.net/zlt2000/article/details/99641821
