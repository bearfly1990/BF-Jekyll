---
layout: post
title: SpringBoot Banner & Log4j2
subtitle: SpringBoot中自定义Banner并使用Log4j2代替默认Logback
date: 2022-04-24
author: BF
thumbnail: /img/bf/java.jpg
catalog: true
toc: true
categories: Diary
header-img: /img/bf/java.jpg
tags:
    - java
    - spring
    - springboot
    - log4j2
---

## Overview

由于自己之前太62了，写的一些东西都删除了。现在重新来弄，不过有时候有些东西明明觉得就是要那样弄的，现在要变着花儿来弄，也挺难受的orz.

这篇配置方式基本来自网上资料的整理。

相关的代码在<https://github.com/bearfly1990/java-playground/tree/main/spring-boot>
<!--more-->

## 参考

- [Spring Boot框架入门教程（快速学习版）](http://c.biancheng.net/spring_boot/)
- [玩转 Spring Boot 的启动 Banner](https://zhuanlan.zhihu.com/p/352966713)
- [log4j2.xml完美配置](https://blog.51cto.com/abcd/2157668)

## Banner.txt

网上有很多文章来讲替换Banner的文章，我这边就是在网上找的一个配置。

字符文字可以去下面这个网站生成：
<http://patorjk.com/software/taag/#p=display&h=0&v=1&f=3D%20Diagonal&t=Type%20Something%20>

```bash
************************************************************************************************
    ${application.title} ${application.version}
    Base on Spring Boot ${spring-boot.version}
************************************************************************************************
System.properties:
-------------------------------------------------------------------------------------------------
    java.specification.version                = ${java.specification.version}
    java.specification.vendor                 = ${java.specification.vendor}
    java.specification.name                   = ${java.specification.name}
    java.vm.specification.version             = ${java.vm.specification.version}
    java.vm.specification.vendor              = ${java.vm.specification.vendor}
    java.vm.specification.name                = ${java.vm.specification.name}
    java.home                                 = ${java.home}
    java.version                              = ${java.version}
    java.vendor                               = ${java.vendor}
    java.vendor.url                           = ${java.vendor.url}
    java.vm.version                           = ${java.vm.version}
    java.vm.vendor                            = ${java.vm.vendor}
    java.vm.name                              = ${java.vm.name}
    java.class.version                        = ${java.class.version}
    java.class.path                           = ${java.class.path}
    java.library.path                         = ${java.library.path}
    java.io.tmpdir                            = ${java.io.tmpdir}
    java.ext.dirs                             = ${java.ext.dirs}
    os.name                                   = ${os.name}
    os.arch                                   = ${os.arch}
    os.version                                = ${os.version}
    user.name                                 = ${user.name}
    user.home                                 = ${user.home}
    user.dir                                  = ${user.dir}

     ██  █████  ██    ██  █████        ██████  ██       █████  ██    ██  ██████  ██████   ██████  ██    ██ ███    ██ ██████
     ██ ██   ██ ██    ██ ██   ██       ██   ██ ██      ██   ██  ██  ██  ██       ██   ██ ██    ██ ██    ██ ████   ██ ██   ██
     ██ ███████ ██    ██ ███████ █████ ██████  ██      ███████   ████   ██   ███ ██████  ██    ██ ██    ██ ██ ██  ██ ██   ██
██   ██ ██   ██  ██  ██  ██   ██       ██      ██      ██   ██    ██    ██    ██ ██   ██ ██    ██ ██    ██ ██  ██ ██ ██   ██
 █████  ██   ██   ████   ██   ██       ██      ███████ ██   ██    ██     ██████  ██   ██  ██████   ██████  ██   ████ ██████

SpringBoot${spring-boot.formatted-version}
====================================================================================================
```

## Spring Boot 日志使用

Spring Boot 默认使用 SLF4J+Logback 记录日志，并提供了默认配置，即使我们不进行任何额外配，也可以使用 SLF4J+Logback 进行日志输出。

我们可以根据自身的需求，通过全局配置文件（application.properties/yml）修改 Spring Boot 日志级别和显示格式等默认配置。

在 application.properties 中，修改 Spring Boot 日志的默认配置，代码如下。

```pro
#日志级别
logging.level.net.biancheng.www=trace
#使用相对路径的方式设置日志输出的位置（项目根目录目录\my-log\mylog\spring.log）
#logging.file.path=my-log/myLog
#绝对路径方式将日志文件输出到 【项目所在磁盘根目录\springboot\logging\my\spring.log】
logging.file.path=/spring-boot/logging
#控制台日志输出格式
logging.pattern.console=%d{yyyy-MM-dd hh:mm:ss} [%thread] %-5level %logger{50} - %msg%n
#日志文件输出格式
logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} === - %msg%n
```

在 Spring Boot 的配置文件 application.porperties/yml 中，可以对日志的一些默认配置进行修改，但这种方式只能修改个别的日志配置，想要修改更多的配置或者使用更高级的功能，则需要通过日志实现框架自己的配置文件进行配置。

Spring 官方提供了各个日志实现框架所需的配置文件，用户只要将指定的配置文件放置到项目的类路径下即可。

| 日志框架                | 配置文件                                                     |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | logback-spring.xml、logback-spring.groovy、logback.xml、logback.groovy |
| Log4j2                  | log4j2-spring.xml、log4j2.xml                                |
| JUL (Java Util Logging) | logging.properties                                           |

我们将 logback.xml、log4j2.xml 等不带 spring 标识的普通日志配置文件，放在项目的类路径下后，这些配置文件会跳过 Spring Boot，直接被日志框架加载。通过这些配置文件，我们就可以达到自定义日志配置的目的。

#### 示例

将 logback.xml 加入到 Spring Boot 项目的类路径下（resources 目录下），该配置文件配置内容如下。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒当scan为true时，此属性生效。默认的时间间隔为1分钟。
debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。
-->
<configuration scan="false" scanPeriod="60 seconds" debug="false">
    <!-- 定义日志的根目录 -->
    <property name="LOG_HOME" value="/app/log"/>
    <!-- 定义日志文件名称 -->
    <property name="appName" value="bianchengbang-spring-boot-logging"></property>
    <!-- ch.qos.logback.core.ConsoleAppender 表示控制台输出 -->
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
   %d表示日期时间，
   %thread表示线程名，
   %-5level：级别从左显示5个字符宽度
   %logger{50} 表示logger名字最长50个字符，否则按照句点分割。
   %msg：日志消息，
   %n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread]**************** %-5level %logger{50} - %msg%n</pattern>
        </layout>
    </appender>

    <!-- 滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件 -->
    <appender name="appLogAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 指定日志文件的名称 -->
        <file>${LOG_HOME}/${appName}.log</file>
        <!--
        当发生滚动时，决定 RollingFileAppender 的行为，涉及文件移动和重命名
        TimeBasedRollingPolicy： 最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。
        -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--
            滚动时产生的文件的存放位置及文件名称 %d{yyyy-MM-dd}：按天进行日志滚动
            %i：当文件大小超过maxFileSize时，按照i进行文件滚动
            -->
            <fileNamePattern>${LOG_HOME}/${appName}-%d{yyyy-MM-dd}-%i.log</fileNamePattern>
            <!--
            可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每天滚动，
            且maxHistory是365，则只保存最近365天的文件，删除之前的旧文件。注意，删除旧文件是，
            那些为了归档而创建的目录也会被删除。
            -->
            <MaxHistory>365</MaxHistory>
            <!--
            当日志文件超过maxFileSize指定的大小是，根据上面提到的%i进行日志文件滚动 注意此处配置SizeBasedTriggeringPolicy是无法实现按文件大小进行滚动的，必须配置timeBasedFileNamingAndTriggeringPolicy
            -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!-- 日志输出格式： -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [ %thread ] ------------------ [ %-5level ] [ %logger{50} : %line ] -
                %msg%n
            </pattern>
        </layout>
    </appender>

    <!--
  logger主要用于存放日志对象，也可以定义日志类型、级别
  name：表示匹配的logger类型前缀，也就是包的前半部分
  level：要记录的日志级别，包括 TRACE < DEBUG < INFO < WARN < ERROR
  additivity：作用在于children-logger是否使用 rootLogger配置的appender进行输出，
  false：表示只用当前logger的appender-ref，true：
  表示当前logger的appender-ref和rootLogger的appender-ref都有效
    -->
    <!-- hibernate logger -->
    <logger name="net.biancheng.www" level="debug"/>
    <!-- Spring framework logger -->
    <logger name="org.springframework" level="debug" additivity="false"></logger>

    <!--
    root与logger是父子关系，没有特别定义则默认为root，任何一个类只会和一个logger对应，
    要么是定义的logger，要么是root，判断的关键在于找到这个logger，然后判断这个logger的appender和level。
    -->
    <root level="info">
        <appender-ref ref="stdout"/>
        <appender-ref ref="appLogAppender"/>
    </root>
</configuration> 
```

Spring Boot 推荐用户使用 logback-spring.xml、log4j2-spring.xml 等这种带有 spring 标识的配置文件。这种配置文件被放在项目类路径后，不会直接被日志框架加载，而是由 Spring Boot 对它们进行解析，这样就可以使用 Spring Boot 的高级功能 Profile，实现在不同的环境中使用不同的日志配置。

更多相关配置可以参考：[Spring Boot日志配置及输出](http://c.biancheng.net/spring_boot/log-config.html)

## Log4j2.xml

为了支持Log4j2，首先要在springboot相关的start中去除掉默认引用的`logback`的包,并加入`spring-boot-starter-log4j2`。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

并在`application.properties`显式中加入`logging.config=classpath:log4j2.xml`，这样有时候忘记去除干净`logback`的包会强制报错。

最后我在网上找了一个感觉不错的**Log4j2.xml**的配置如下，其中有很详细的配置解释。另外本来其中没有**Debug**级别的配置，我加了一下。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration后面的status,这个用于设置log4j2自身内部的信息输出,可以不设置,当设置成trace时,你会看到log4j2内部各种详细输出-->
<!--monitorInterval：Log4j能够自动检测修改配置 文件和重新配置本身,设置间隔秒数-->
<configuration status="WARN" monitorInterval="1800">

    <Properties>
        <!-- ==============================================公共配置============================================== -->
        <!-- 设置日志文件的目录名称 -->
        <property name="logFileName">spring-boot</property>

        <!-- 日志默认存放的位置,可以设置为项目根路径下,也可指定绝对路径 -->
        <!-- 存放路径一:通用路径,window平台 -->
        <!-- <property name="basePath">d:/logs/${logFileName}</property> -->
        <!-- 存放路径二:web工程专用,java项目没有这个变量,需要删掉,否则会报异常,这里把日志放在web项目的根目录下 -->
        <!-- <property name="basePath">${web:rootDir}/${logFileName}</property> -->
        <!-- 存放路径三:web工程专用,java项目没有这个变量,需要删掉,否则会报异常,这里把日志放在tocmat的logs目录下 -->
        <property name="basePath">log/${logFileName}</property>

        <!-- 控制台默认输出格式,"%-5level":日志级别,"%l":输出完整的错误位置,是小写的L,因为有行号显示,所以影响日志输出的性能 -->
        <property name="console_log_pattern">%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] %l - %m%n</property>
        <!-- 日志文件默认输出格式,不带行号输出(行号显示会影响日志输出性能);%C:大写,类名;%M:方法名;%m:错误信息;%n:换行 -->
        <!-- <property name="log_pattern">%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] %C.%M - %m%n</property> -->
        <!-- 日志文件默认输出格式,另类带行号输出(对日志输出性能未知);%C:大写,类名;%M:方法名;%L:行号;%m:错误信息;%n:换行 -->
        <property name="log_pattern">%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level] %C.%M[%L line] - %m%n</property>

        <!-- 日志默认切割的最小单位 -->
        <property name="every_file_size">20MB</property>
        <!-- 日志默认输出级别 -->
        <property name="output_log_level">DEBUG</property>

        <!-- ===========================================所有级别日志配置=========================================== -->
        <!-- 日志默认存放路径(所有级别日志) -->
        <property name="rolling_fileName">${basePath}/all.log</property>
        <!-- 日志默认压缩路径,将超过指定文件大小的日志,自动存入按"年月"建立的文件夹下面并进行压缩,作为存档 -->
        <property name="rolling_filePattern">${basePath}/%d{yyyy-MM}/all-%d{yyyy-MM-dd-HH}-%i.log.gz</property>
        <!-- 日志默认同类型日志,同一文件夹下可以存放的数量,不设置此属性则默认为7个,filePattern最后要带%i才会生效 -->
        <property name="rolling_max">500</property>
        <!-- 日志默认同类型日志,多久生成一个新的日志文件,这个配置需要和filePattern结合使用;
          如果设置为1,filePattern是%d{yyyy-MM-dd}到天的格式,则间隔一天生成一个文件
          如果设置为12,filePattern是%d{yyyy-MM-dd-HH}到小时的格式,则间隔12小时生成一个文件 -->
        <property name="rolling_timeInterval">12</property>
        <!-- 日志默认同类型日志,是否对封存时间进行调制,若为true,则封存时间将以0点为边界进行调整,
          如:现在是早上3am,interval是4,那么第一次滚动是在4am,接着是8am,12am...而不是7am -->
        <property name="rolling_timeModulate">true</property>

        <!-- ============================================Debug级别日志============================================ -->
        <!-- Debug日志默认存放路径(Info级别日志) -->
        <property name="debug_fileName">${basePath}/debug.log</property>
        <!-- Debug日志默认压缩路径,将超过指定文件大小的日志,自动存入按"年月"建立的文件夹下面并进行压缩,作为存档 -->
        <property name="debug_filePattern">${basePath}/%d{yyyy-MM}/debug-%d{yyyy-MM-dd}-%i.log.gz</property>
        <!-- Debug日志默认同一文件夹下可以存放的数量,不设置此属性则默认为7个 -->
        <property name="debug_max">100</property>
        <!-- 日志默认同类型日志,多久生成一个新的日志文件,这个配置需要和filePattern结合使用;
          如果设置为1,filePattern是%d{yyyy-MM-dd}到天的格式,则间隔一天生成一个文件
          如果设置为12,filePattern是%d{yyyy-MM-dd-HH}到小时的格式,则间隔12小时生成一个文件 -->
        <property name="debug_timeInterval">1</property>
        <!-- 日志默认同类型日志,是否对封存时间进行调制,若为true,则封存时间将以0点为边界进行调整,
          如:现在是早上3am,interval是4,那么第一次滚动是在4am,接着是8am,12am...而不是7am -->
        <property name="debug_timeModulate">true</property>


        <!-- ============================================Info级别日志============================================ -->
        <!-- Info日志默认存放路径(Info级别日志) -->
        <property name="info_fileName">${basePath}/info.log</property>
        <!-- Info日志默认压缩路径,将超过指定文件大小的日志,自动存入按"年月"建立的文件夹下面并进行压缩,作为存档 -->
        <property name="info_filePattern">${basePath}/%d{yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log.gz</property>
        <!-- Info日志默认同一文件夹下可以存放的数量,不设置此属性则默认为7个 -->
        <property name="info_max">100</property>
        <!-- 日志默认同类型日志,多久生成一个新的日志文件,这个配置需要和filePattern结合使用;
          如果设置为1,filePattern是%d{yyyy-MM-dd}到天的格式,则间隔一天生成一个文件
          如果设置为12,filePattern是%d{yyyy-MM-dd-HH}到小时的格式,则间隔12小时生成一个文件 -->
        <property name="info_timeInterval">1</property>
        <!-- 日志默认同类型日志,是否对封存时间进行调制,若为true,则封存时间将以0点为边界进行调整,
          如:现在是早上3am,interval是4,那么第一次滚动是在4am,接着是8am,12am...而不是7am -->
        <property name="info_timeModulate">true</property>

        <!-- ============================================Warn级别日志============================================ -->
        <!-- Warn日志默认存放路径(Warn级别日志) -->
        <property name="warn_fileName">${basePath}/warn.log</property>
        <!-- Warn日志默认压缩路径,将超过指定文件大小的日志,自动存入按"年月"建立的文件夹下面并进行压缩,作为存档 -->
        <property name="warn_filePattern">${basePath}/%d{yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log.gz</property>
        <!-- Warn日志默认同一文件夹下可以存放的数量,不设置此属性则默认为7个 -->
        <property name="warn_max">100</property>
        <!-- 日志默认同类型日志,多久生成一个新的日志文件,这个配置需要和filePattern结合使用;
          如果设置为1,filePattern是%d{yyyy-MM-dd}到天的格式,则间隔一天生成一个文件
          如果设置为12,filePattern是%d{yyyy-MM-dd-HH}到小时的格式,则间隔12小时生成一个文件 -->
        <property name="warn_timeInterval">1</property>
        <!-- 日志默认同类型日志,是否对封存时间进行调制,若为true,则封存时间将以0点为边界进行调整,
          如:现在是早上3am,interval是4,那么第一次滚动是在4am,接着是8am,12am...而不是7am -->
        <property name="warn_timeModulate">true</property>

        <!-- ============================================Error级别日志============================================ -->
        <!-- Error日志默认存放路径(Error级别日志) -->
        <property name="error_fileName">${basePath}/error.log</property>
        <!-- Error日志默认压缩路径,将超过指定文件大小的日志,自动存入按"年月"建立的文件夹下面并进行压缩,作为存档 -->
        <property name="error_filePattern">${basePath}/%d{yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log.gz</property>
        <!-- Error日志默认同一文件夹下可以存放的数量,不设置此属性则默认为7个 -->
        <property name="error_max">100</property>
        <!-- 日志默认同类型日志,多久生成一个新的日志文件,这个配置需要和filePattern结合使用;
          如果设置为1,filePattern是%d{yyyy-MM-dd}到天的格式,则间隔一天生成一个文件
          如果设置为12,filePattern是%d{yyyy-MM-dd-HH}到小时的格式,则间隔12小时生成一个文件 -->
        <property name="error_timeInterval">1</property>
        <!-- 日志默认同类型日志,是否对封存时间进行调制,若为true,则封存时间将以0点为边界进行调整,
          如:现在是早上3am,interval是4,那么第一次滚动是在4am,接着是8am,12am...而不是7am -->
        <property name="error_timeModulate">true</property>

        <!-- ============================================控制台显示控制============================================ -->
        <!-- 控制台显示的日志最低级别 -->
        <property name="console_print_level">DEBUG</property>

    </Properties>

    <!--定义appender -->
    <appenders>
        <!-- =======================================用来定义输出到控制台的配置======================================= -->
        <Console name="Console" target="SYSTEM_OUT">
            <!-- 设置控制台只输出level及以上级别的信息(onMatch),其他的直接拒绝(onMismatch)-->
            <ThresholdFilter level="${console_print_level}" onMatch="ACCEPT" onMismatch="DENY"/>
            <!-- 设置输出格式,不设置默认为:%m%n -->
            <PatternLayout pattern="${console_log_pattern}"/>
        </Console>

        <!-- ================================打印root中指定的level级别以上的日志到文件================================ -->
        <RollingFile name="RollingFile" fileName="${rolling_fileName}" filePattern="${rolling_filePattern}">
            <PatternLayout pattern="${log_pattern}"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="${rolling_timeInterval}" modulate="${warn_timeModulate}"/>
                <SizeBasedTriggeringPolicy size="${every_file_size}"/>
            </Policies>
            <!-- 设置同类型日志,同一文件夹下可以存放的数量,如果不设置此属性则默认存放7个文件 -->
            <DefaultRolloverStrategy max="${rolling_max}" />
        </RollingFile>
        <!-- =======================================打印DEBUG级别的日志到文件======================================= -->
        <RollingFile name="DebugFile" fileName="${debug_fileName}" filePattern="${debug_filePattern}">
            <PatternLayout pattern="${log_pattern}"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="${debug_timeInterval}" modulate="${debug_timeModulate}"/>
                <SizeBasedTriggeringPolicy size="${every_file_size}"/>
            </Policies>
            <DefaultRolloverStrategy max="${debug_max}" />
            <Filters>
                <ThresholdFilter level="INFO" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="DEBUG" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
        </RollingFile>

        <!-- =======================================打印INFO级别的日志到文件======================================= -->
        <RollingFile name="InfoFile" fileName="${info_fileName}" filePattern="${info_filePattern}">
            <PatternLayout pattern="${log_pattern}"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="${info_timeInterval}" modulate="${info_timeModulate}"/>
                <SizeBasedTriggeringPolicy size="${every_file_size}"/>
            </Policies>
            <DefaultRolloverStrategy max="${info_max}" />
            <Filters>
                <ThresholdFilter level="WARN" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="INFO" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
        </RollingFile> 

        <!-- =======================================打印WARN级别的日志到文件======================================= -->
        <RollingFile name="WarnFile" fileName="${warn_fileName}" filePattern="${warn_filePattern}">
            <PatternLayout pattern="${log_pattern}"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="${warn_timeInterval}" modulate="${warn_timeModulate}"/>
                <SizeBasedTriggeringPolicy size="${every_file_size}"/>
            </Policies>
            <DefaultRolloverStrategy max="${warn_max}" />
            <Filters>
                <ThresholdFilter level="ERROR" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="WARN" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
        </RollingFile>

        <!-- =======================================打印ERROR级别的日志到文件======================================= -->
        <RollingFile name="ErrorFile" fileName="${error_fileName}" filePattern="${error_filePattern}">
            <PatternLayout pattern="${log_pattern}"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="${error_timeInterval}" modulate="${error_timeModulate}"/>
                <SizeBasedTriggeringPolicy size="${every_file_size}"/>
            </Policies>
            <DefaultRolloverStrategy max="${error_max}" />
            <Filters>
                <ThresholdFilter level="FATAL" onMatch="DENY" onMismatch="NEUTRAL"/>
                <ThresholdFilter level="ERROR" onMatch="ACCEPT" onMismatch="DENY"/>
            </Filters>
        </RollingFile>
    </appenders>

    <!--定义logger,只有定义了logger并引入的appender,appender才会生效-->
    <loggers>
        <!-- 设置打印sql语句配置开始,以下两者配合使用,可以优化日志的输出信息,减少一些不必要信息的输出 -->
        <!-- 设置java.sql包下的日志只打印DEBUG及以上级别的日志,此设置可以支持sql语句的日志打印 -->
        <logger name="java.sql" level="DEBUG" additivity="false">
            <appender-ref ref="Console"/>
        </logger>
        <!-- 设置org.mybatis.spring包下的日志只打印WARN及以上级别的日志 -->
        <logger name="org.mybatis.spring" level="WARN" additivity="false">
            <appender-ref ref="Console"/>
        </logger>
        <!-- 设置org.springframework包下的日志只打印WARN及以上级别的日志 -->
        <logger name="org.springframework" level="WARN" additivity="false">
            <appender-ref ref="Console"/>
        </logger>
        <!-- 设置com.qfx.workflow.service包下的日志只打印WARN及以上级别的日志 -->
        <logger name="com.qfx.workflow.service" level="WARN" additivity="false">
            <appender-ref ref="Console"/>
        </logger>
        <!-- 设置打印sql语句配置结束 -->

        <!--建立一个默认的root的logger-->
        <root level="${output_log_level}">
            <appender-ref ref="RollingFile"/>
            <appender-ref ref="Console"/>
            <appender-ref ref="DebugFile"/>
            <appender-ref ref="InfoFile"/>
            <appender-ref ref="WarnFile"/>
            <appender-ref ref="ErrorFile"/>
        </root>
    </loggers>

</configuration>
```

最后的输出结果如下，很好的将不同级别的Log输出：

![image-20220424205336248](/img/post/2022/04/2022-04-24-springboot-banner-log4j2/log-output.png)

## Console颜色

有时候需要在VM中加参数，才能在Console中支持有颜色的输出。
{% raw %}
```bash
%clr{%d{yyyy-MM-dd HH:mm:ss,SSS}}{magenta} %clr{%pid}{blue} %clr{%5p} -&#45;&#45; %cyan{%l} -&#45;&#45; %m %n
%highlight{%date{HH:mm:ss.SSS} [%thread] %-5level %logger{36}  - %msg%n}{FATAL=red, ERROR=red, WARN=yellow, INFO=cyan, DEBUG=cyan,TRACE=blue}
```
{% endraw %}

```bash
-Dspring.output.ansi.enabled=ALWAYS
-Dlog4j.skipJansi=false
```
