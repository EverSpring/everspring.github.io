---
title: 关于Logback+MyBatis日志输出的一些思考
date: 2022-09-29 12:25:00
author: EverSpring
top: true
toc: false
mathjax: false
categories: JAVA
tags:
  - 日志
---
# 一.环境

MyBatis-Plus：3.4.3.4

SpringBoot：2.5.8

Logback：1.2.9（SpringBoot自动依赖）

# 二.问题

1. sql语句和业务日志合在一起，sql太多会影响业务日志分析，因为平时不需要看，出问题的时候才用上，所以sql日志和业务日志应该分成不同的文件记录
2. 有些场景需要开发的时候同时在IDE控制台打印sql执行语句、输出sql到日志文件中
3. 日志分开后，又会导致查问题时不容易分清楚是否是同一次请求交易，简单通过线程名、时间点来区分也不方便而且容易出错
4. 有时数据会因为事务等原因导致写入失效，需要查看事务日志
5. 业务模块很多的单体应用，所以日志糅杂在一起，查看不方便，按照业务模块记录到不同的日志文件中

# 三.解答

先贴出logback.xml的配置，后面的解答就是此配置文件中的内容

<details>

<summary><b>展开查看logback配置</b></summary>

<pre>

<code>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <property name="logger.path" value="/home/logs" />

    <!-- 彩色日志 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}-%line){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

    <!--输出到控制台-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>DEBUG</level>
        </filter>
        <encoder>
            <Pattern>${CONSOLE_LOG_PATTERN}</Pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--输出到文件-->
    <!-- 时间滚动输出 level为 DEBUG 日志 -->
    <appender name="DEBUG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${logger.path}/log_debug.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}-%line - %m%n</pattern>
            <charset>UTF-8</charset> <!-- 设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志归档 -->
            <fileNamePattern>${logger.path}/debug/log-debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录debug级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>debug</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 时间滚动输出 level为 INFO 日志 -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${logger.path}/log_info.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}-%line - %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>${logger.path}/info/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录info级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 时间滚动输出 level为 WARN 日志 -->
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${logger.path}/log_warn.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}-%line - %m%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logger.path}/warn/log-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录warn级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>


    <!-- 时间滚动输出 level为 ERROR 日志 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${logger.path}/log_error.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}-%line - %m%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${logger.path}/error/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录ERROR级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    <!--
        root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
        不能设置为INHERITED或者同义词NULL。默认是DEBUG
        可以包含零个或多个元素，标识这个appender将会添加到这个logger。
    -->
    <!-- 打印sql，但不打印事务等 com.bank.mapper为我自己mapper存放的包名-->
    <logger name="com.bank.mapper" level="DEBUG">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="DEBUG_FILE"/>
    </logger>
    <!-- 打印事务、mapper注册等信息，打印到控制台、记录到debug_file -->
    <logger name="org.mybatis.spring.SqlSessionUtils" level="DEBUG">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="DEBUG_FILE"/>
    </logger>

    <root level="info">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="DEBUG_FILE" />
        <appender-ref ref="INFO_FILE" />
        <appender-ref ref="WARN_FILE" />
        <appender-ref ref="ERROR_FILE" />
    </root>

</configuration>
```
</code>
</pre>
</details>

1. 把业务日志代码记录到info级别的log中，即配置中的log_info.log；sql日志和事务日志记录到debug级别中，即log_debug.log。通过两个不同的appender实现。
2. MyBatis-Plus的配置中默认支持打印日志到控制台或者输出到日志文件，但不能同时支持，要么只在控制台打印，要么只输出到日志文件。默认配置即可能以下配置实现：

```
mybatis-plus:
  configuration:
    logImpl: org.apache.ibatis.logging.slf4j.Slf4jImpl # 默认输出到日志文件
    logImpl: org.apache.ibatis.logging.stdout.StdOutImpl #默认输出到控制台
```

如果严格区分环境，一般是不需要同时输出的。

解决方式，通过logback配置处理

- a) 控制台的appender的filter level为debug，因为mybatis代码中事务打印和sql打印都是debug级别；
- b)指定`<logger>`配置。
  ```
  <logger name="com.bank.mapper" level="DEBUG">
          <appender-ref ref="CONSOLE"/>
          <appender-ref ref="DEBUG_FILE"/>
      </logger>
      <!-- 打印事务、mapper注册等信息，打印到控制台、记录到debug_file -->
      <logger name="org.mybatis.spring.SqlSessionUtils" level="DEBUG">
          <appender-ref ref="CONSOLE"/>
          <appender-ref ref="DEBUG_FILE"/>
      </logger>
  ```
  
  其中`org.mybatis.spring.SqlSessionUtils`是事务开启、关闭的日志打印类，如果还需打印mapper的注册信息等，可以按package打印`org.mybatis.spring`。以下为事务打印的实现方法，可以看出都是debug级别的
  ```
  if (environment.getTransactionFactory() instanceof SpringManagedTransactionFactory) {
          LOGGER.debug(() -> "Registering transaction synchronization for SqlSession [" + session + "]");
  
          holder = new SqlSessionHolder(session, executorType, exceptionTranslator);
          TransactionSynchronizationManager.bindResource(sessionFactory, holder);
          TransactionSynchronizationManager
              .registerSynchronization(new SqlSessionSynchronization(holder, sessionFactory));
          holder.setSynchronizedWithTransaction(true);
          holder.requested();
        } else {
          if (TransactionSynchronizationManager.getResource(environment.getDataSource()) == null) {
            LOGGER.debug(() -> "SqlSession [" + session
                + "] was not registered for synchronization because DataSource is not transactional");
          } else {
            throw new TransientDataAccessResourceException(
                "SqlSessionFactory must be using a SpringManagedTransactionFactory in order to use Spring transaction synchronization");
          }
        }
      } else {
        LOGGER.debug(() -> "SqlSession [" + session
            + "] was not registered for synchronization because synchronization is not active");
      }
  ```

3. 关于sql日志和业务日志分成不同文件，日志查看连贯性受影响，可以通过MDC处理，见我另一篇文件[MDC在多线程中的使用](https://everspring.github.io/2022/08/23/mdc-xiang-jie/)，写好MDC实现后，在日志中配置好即可。
也可引用轻量日志组件[TLog](https://gitee.com/dromara/TLog.git)，就可以不用再自己实现MDC，参考文章：[TLog MDC模式下logback的配置](https://tlog.yomahub.com/pages/eea781/)

4. Logback的`<appender>`+`<logger>`可以很好的解决按照不同模块记录日志。有两种方式：
* a) 工程按照业务模块划分成了不同package，可以通过`<appender>`+`<logger name="包名">`组合实现，此方式不需要修改java代码；
* b) 如果不能按照package划分，就需要通过代码中`Logger LOGGER = LoggerFactory.getLogger("logger name的值");`来指定（如用了Lombok，可用`@Slf4j(topic="logger name的值")`）
```xml
    <!-- 时间滚动输出 level为 pbc模块INFO 日志 -->
    <appender name="PBC_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${logger.path}/pbc_log_info.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %C.%M-%line - %m%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>${logger.path}/info/pbc_log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录info级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 业务模块 -->
    <logger name="pbc_module" level="info" additivity="false">
        <appender-ref ref="PBC_LOG"/>
    </logger>
```