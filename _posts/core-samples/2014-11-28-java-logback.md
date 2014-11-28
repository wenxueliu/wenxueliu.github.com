---
layout: post
category : java
tagline: "java 日志系统"
tags : [ lockback, slf4j, log, log4j ]
---
{% include JB/setup %}

写代码中的日志是一个除了用代码实现功能之外最基础最基础的一个技能了，是一个必须掌握的技能。但是目前为止，关于如何日志的文章和书籍还是不多。

##写日志的必要性

碰到QA提的一个bug的时候，我见识过两种方式的答复：a)请给我重现步骤和重现数据；b）把当时的日志给我。答复前者的，一般需要花很多时间去找问题出现在那里，如果是别人开发的模块的话，花费的时间更多。答复后者的，一般能很快的找到出问题的点，然后就可以开始进入修复的流程。

从概念上来说：日志是一个可运行的、可维护的软件的基础组成部分；通过日志，我们可以了解软件系统在运行中的实时状态，历史状态和异常状态等。一个没有良好日志的软件是所有人的噩梦。

如果你不想给自己找麻烦，你还是把日志好好写写。 

##为什么选Logback？ 


##如何配置Logback

下面会简单介绍一下Logback的配置，适用于开始配置和开始入门，适用于一般情况下的使用，如果想了解更多的信息，建议看看[Logback的官方文档](http://logback.qos.ch/manual/introduction.html) 


###Logback简单介绍 

简单来说就是Log4j 1流行了，发现有一些问题是无法解决的，于是又出来了Logback，在Log4j的基础上提升了性能，提高了功能等等。不过前几天有出来了Log4j 2，据说是相对于Logbak来说又提升了性能提高了功能。 


###关于SLF4J和Logback 

SLF4J(slf4j.org)又称Simple Logging Facade for Java，是一个通用的logging接口，它试图一统Logging框架的天下，兼容了(Log4j 1, java.util.logging和Jakarta Commons Logging）这三个最流行的Logging框架。Logback就是SLF4J的默认实现。 

###依赖包导入

下文以Logback 1.1.2及slf4j1.7.6版本为例子。一般情况下，你按照我下面说的就可以了，如果不行的话，你可以去翻翻英文的文档：http://logback.qos.ch/setup.html。 


###一般程序 

####Maven版

	<dependency>
		<groupId>ch.qos.logback</groupId>
		<artifactId>logback-classic</artifactId>
		<version>1.1.2</version>
	</dependency>

####非Maven版

下载[logback](http://logback.qos.ch/dist/logback-1.1.2.zip)之后，在其根目录下有logback-core-1.1.2.jar和logback-classic-1.1.2.jar这俩个Jar，在logback-examples\lib下有slf4j-api-1.7.6.jar这个jar,把这三个Jar添加到你的代码包路径中。 


###非独立运行程序

如果你做的是一个Lib或者API，那么你就不应该依赖于具体的slf4j实现。所以你对logback的依赖应该是在运行测试代码的时候，具体实现方式如下文所示：

####Maven版

	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-api</artifactId>
		<version>1.7.6</version>
	</dependency>
	<dependency>
		<groupId>ch.qos.logback</groupId>
		<artifactId>logback-classic</artifactId>
		<version>1.1.2</version>
		<scope>test</scope>
	</dependency>
	
	
当然了你是要把版本信息放在pom.xml中的<dependencies>还是<dependencyManagement>里面就你自己决定吧。 

####非Maven版

在发布的时候不要把只有Logback才用到jar打包进你的发布程序里面。（如果觉得绕口再多读两遍） 


###与遗留Logging框架兼容

目前行业除了Logback之外，广泛使用的还有其他四种Logging框架：

    Log4j 1 (http://logging.apache.org/log4j/1.2/)
    Log4j 2 (http://logging.apache.org/log4j/2.x/)
    java.util.logging (http://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html)
    Apache commons Logging (http://commons.apache.org/proper/commons-logging/)

Log4j 2因为是刚出来的，目前SLF4J对其的兼容性还未知，对于其他的三种框架，SLF4J都提供了兼容性的支持。下面介绍了如何让Logbak兼容这些框架，另外，也可以阅读官方说明：http://www.slf4j.org/legacy.html 


##Logback 配置文件简介 

 Logback 会按照如下的顺序在classpath中读取配置文件，如果读取到任何一个，则停止继续寻找。

* logback.groovy 这个是使用groovy语法的配置文件
* logback.-test.xml
* logback.xml

如果以上三个文件都没能在classpath中找到，则会使用默认配置。默认配置如下：

	输出格式 %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n  
   	输出方向：System.out
    输出级别：Debug

在我的经验中，基本上按照如下策略来配置Logback文件： 

###Maven项目

* 在src/test/resources目录中放置logback-test.xml，该配置文件中log的输出是到输出到console，对于本应用的代码是debug级别，对于其他的是info级别。 
* 在src/main/resources目录中放置logback.xml，该配置文件中log的输出是输出到文件中，该文件每日滚动压缩打包备份，对于本应用的代码是info级别，对于其他的是warn级别。 

###配置示例

####logback.xml

	<configuration scan="true" scanPeriod="30 seconds">
		<!--Appendar详解: http://logback.qos.ch/manual/appenders.html#RollingFileAppender -->
		<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
		    <!-- 当前Log文件名 -->
		    <file>ldap-pwd.log</file>
		    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
		        <!-- 非当天的Log文件压缩备份为 archive/ldap-pwd.2014-08-10.zip -->
		        <fileNamePattern>archive/ldap-pwd.%d{yyyy-MM-dd}.zip</fileNamePattern>
		        <!-- 超过30天的备份文件会被删除 -->
		        <maxHistory>30</maxHistory>
		    </rollingPolicy>

		    <layout class="ch.qos.logback.classic.PatternLayout">
		        <!-- 格式说明:http://logback.qos.ch/manual/layouts.html#ClassicPatternLayout -->
		        <Pattern>%d [%thread] %-5level %40logger{40} - %msg%n</Pattern>
		    </layout>
		</appender>

		<logger name="cn.justfly.training.logging" level="info" />

		<root level="warn">
		    <appender-ref ref="FILE" />
		</root>
	</configuration>

####logback-test.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<configuration>
		<appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
		    <encoder>
		        <pattern>%d [%thread] %-5level %40logger{40} - %msg%n</pattern>
		    </encoder>
		</appender>
		<logger name="cn.justfly.training.logging" level="debug" />

		<root level="info">
		    <appender-ref ref="Console" />
		</root>
	</configuration>




##开发实例

使用Logging框架写Log基本上就三个步骤

* 引入loggerg类和logger工厂类
* 声明logger
* 记录日志
  

	//引入slf4j接口的Logger和LoggerFactory
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;

	public class UserService {
	  //声明一个Logger，这个是static的方式，我比较习惯这么写。
	  private final static Logger logger = LoggerFactory.getLogger(UserService.class);

	  public boolean verifyLoginInfo(String userName, String password) {
		// log it，输出的log信息将会是："Start to verify User [Justfly]
		logger.info("Start to verify User [{}]", userName);
		return false;
	  }
	}
	

全部 logger 格式 参考[这里](http://www.slf4j.org/apidocs/index.html)


###格式1

	logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));

更好的方式是，尤其是里面的参数非常多或构造参数需要调用 toString 等函数
	
	if(logger.isDebugEnabled()) {
	  	logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
	}

###格式2

	logger.debug("The new entry is "+entry+".");
	
更好的方式是

	logger.debug("The new entry is {}.", entry);

	
logger.debug("Value {} was inserted between {} and {}.", new Bytes[]{newVal, below, above});
	
###格式3

	logger.debug("{}", complexObject.toString());
	
更好的方式是
	
	logger.debug("{}", complexObject.toString());
	

原因解释在[这里](http://www.slf4j.org/apidocs/index.html)
	
###注2
	
关于Logger对象是否要声明为静态的业界有过一些讨论，Logback的作者最早是推荐使用对象变量的方式来声明，后来他自己也改变了想法。想详细了解的同学可以去z看[这里](http://slf4j.org/faq.html#declared_static)
两种方式的优劣概述如下：

* 静态Logger对象相对来说更符合语义，节省CPU，节省内存，不支持注入
* 对象变量Logger支持注入，对于一个JVM中运行的多个引用了同一个类库的应用程序，可以在不同的应用程序中对同个类的Logger进行不同的配置。比如Tomcat上部署了俩个应用，他们都引用了同一个lib。



##日志级别

很多人对日志级别没有理解清楚，导致输出混乱，不便于调试，如何判读日志输出级别，是需要非常仔细考量的。

SLF4J把Log分成了Error，Warn，Info，Debug和Trace五个级别。我们可以把这俩个级别分成两组

###用户级别

Error、Warn和Info这三个级别的Log会出现在生产环境上，他们必须是运维人员能阅读明白的

####Error

影响到程序正常运行、当前请求正常运行的异常情况,例如：

        打开配置文件失败
        第三方应用网络连接异常
        SQLException
        
不应该出现的情况，例如：
        
        某个Service方法返回的List里面应该有元素的时候缺获得一个空List
        做字符转换的时候居然报错说没有GBK字符集

####Warn

不应该出现但是不影响程序、当前请求正常运行的异常情况，例如：

        有容错机制的时候出现的错误情况
        找不到配置文件，但是系统能自动创建配置文件

即将接近临界值的时候，例如：

        缓存池占用达到警告线

####Info

系统运行信息
       
        Service方法的出入口
        主要逻辑中的分步骤

外部接口部分
        
        客户端请求参数和返回给客户端的结果
        调用第三方时的调用参数和调用结果

###开发级别

Debug和Trace这俩个级别主要是在开发期间使用或者当系统出现问题后开发人员介入调试的时候用的，需要有助于提供详细的信息。

####Debug

简要记录的程序运行的每一个步骤。

####Trace

主要用于记录系统运行中的每一个步骤的完整信息。

##Log中的要点

###Log上下文

在Log中必须尽量带入上下文的信息，对比以下俩个Log信息，后者比前者更有作用

    "开始导入配置文件"
    "开始导入配置文件[/etc/myService/config.properties]"
    
###考虑Log的读者

对于用户级别的Log，它的读者可能是使用了你的框架的其他开发者，可能是运维人员，可能是普通用户。你需要尽量以他们可以理解的语言来组织Log信息，如果你的Log能对他们的使用提供有用的帮助就更好了。
下面的两条Log中，前者对于非代码维护人员的帮助不大，后者更容易理解。

    "开始执行getUserInfo 方法，用户名[jimmy]"
    "开始获取用户信息，用户名[jimmy]"

下面的这个Log对于框架的使用者提供了极大的帮助

    "无法解析参数[12 03, 2013]，birthDay参数需要符合格式[yyyy-MM-dd]"
    
###Log中的变量用[]与普通文本区分开来(可以为自己希望的其他符号)

把变量和普通文本隔离有这么几个作用

    在你阅读Log的时候容易捕捉到有用的信息
    在使用工具分析Log的时候可以更方便抓取
    在一些情况下不容易混淆

对比以下下面的两条Log，前者发生了混淆：

    "获取用户lj12月份发邮件记录数"
    "获取用户[lj1][2]月份发邮件记录数" 
 
###Error或者Warn级别中碰到Exception的情况尽量log 完整的异常信息
 
Error和Warn级别是比较严重的情况，意味着系统出错或者危险，我们需要更多的信息来帮助分析原因，这个时候越多的信息越有帮助。这个时候最好的做法是Log以下全部内容：

* 你是在做什么事情的时候出错了
* 你是在用什么数据做这个事情的时候出错了
* 出错的信息是什么

对比下面三个Log语句，第一个提供了详尽的信息，第二个只提供了部分信息，Exception的Message不一定包含有用的信息，第三个只告诉你出错了，其他的你一无所知。

* log.error("获取用户[{}]的用户信息时出错",userName,ex);
* log.error("获取用户[{}]的用户信息时报错，错误信息：[{}]",userName,ex.getMessage());
* log.error("获取用户信息时出错");


###对于Exception，要每次都Log StackTrace吗？

在一些Exception处理机制中，我们会每层或者每个Service对应一个RuntimeException类，并把他们抛出去，留给最外层的异常处理层处理。典型代码如下:

	try{
	  
	}catch(Exception ex){
	  	String errorMessage=String.format("Error while reading information of user [%s]",userName);
	  	logger.error(errorMessage,ex);
	  	throw new UserServiceException(errorMessage,ex);
	}

这个时候问题来了，在最底层出错的地方 Log 了异常的StackTrace，在你把这个异常想上层抛的过程中，在最外层的异常处理层的时候，还会再 Log 一次异常的StackTrace，这样子你的Log中会有大篇的重复信息。

我碰到这种情况一般是这么处理的：Log之！原因有以下这几个方面：

* 这个信息很重要，我不确认再往上的异常处理层中是否会正常的把它的StackTrace打印出来。
* 如果这个异常信息在往上传递的过程中被多次包装，到了最外层打印StackTrace的时候最底层的真正有用的出错原因有可能不会被打印出来。
* 如果有人改变了LogbackException打印的配置，使得不能完全打印的时候，这个信息可能就丢了。
* 就算重复了又怎么样？都Error了都Warning了还省那么一点空间吗？

###参考

http://www.blogjava.net/justfly/archive/2014/08/13/416925.html
http://www.blogjava.net/justfly/archive/2014/08/13/416925.html
http://www.slf4j.org/docs.html

