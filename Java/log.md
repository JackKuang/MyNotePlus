# 常用日志框架（log4j，slf4j，logback）有啥区别

## 一、概述

对于一个应用程序来说日志记录是必不可少的一部分。线上问题追踪，基于日志的业务逻辑统计分析等都离不日志。java领域存在多种日志框架，目前常用的日志框架包括Log4j 1，Log4j 2，Commons Logging，Slf4j，Logback，Jul。

其中主要的日志类别如下：

* **Commons Logging**
  * Apache基金会所属的项目，是一套Java日志接口，之前叫Jakarta Commons Logging，后更名为Commons Logging。
* **Slf4j**
  * 类似于Commons Logging，是一套简易Java日志门面，本身并无日志的实现。（Simple Logging Facade for Java，缩写Slf4j）。

* **Log4j**
  * Apache Log4j是一个基于Java的日志记录工具，是Apache软件基金会的一个项目。 Log4j是几种Java日志框架之一。
* **Log4j2**
  * Apache Log4j 2是apache开发的一款Log4j的升级产品。
* **Logback**
  *  logback是由log4j创始人设计的又一个开源日志组件，作为流行的log4j项目的后续版本，从而替代log4j。



**日志历史**

> - 1996年早期，欧洲安全电子市场项目组决定编写它自己的程序跟踪API(Tracing API)。经过不断的完善，这个API终于成为一个十分受欢迎的Java日志软件包，即Log4j。后来Log4j成为Apache基金会项目中的一员。
>
> - 期间Log4j近乎成了Java社区的日志标准。据说Apache基金会还曾经建议Sun引入Log4j到java的标准库中，但Sun拒绝了。
>
> - 2002年Java1.4发布，Sun推出了自己的日志库JUL(Java Util Logging),其实现基本模仿了Log4j的实现。在JUL出来以前，Log4j就已经成为一项成熟的技术，使得Log4j在选择上占据了一定的优势。
>
> - 接着，Apache推出了Jakarta Commons Logging，JCL只是定义了一套日志接口(其内部也提供一个Simple Log的简单实现)，支持运行时动态加载日志组件的实现，也就是说，在你应用代码里，只需调用Commons Logging的接口，底层实现可以是Log4j，也可以是Java Util Logging。
>
> - 后来(2006年)，Ceki Gülcü不适应Apache的工作方式，离开了Apache。然后先后创建了Slf4j(日志门面接口，类似于Commons Logging)和Logback(Slf4j的实现)两个项目，并回瑞典创建了QOS公司，QOS官网上是这样描述Logback的：The Generic，Reliable Fast&Flexible Logging Framework(一个通用，可靠，快速且灵活的日志框架)。
>
> - 现今，Java日志领域被划分为两大阵营：Commons Logging阵营和Slf4j阵营。
>   Commons Logging在Apache大树的笼罩下，有很大的用户基数。但有证据表明，形式正在发生变化。2013年底有人分析了GitHub上30000个项目，统计出了最流行的100个Libraries，可以看出Slf4j的发展趋势更好：
>
>   ![java_populor_jar](https://cnblogpic.oss-cn-qingdao.aliyuncs.com/blogpic/java_log/java_populor_jar.png)
>
> - Apache眼看有被Logback反超的势头，于2012-07重写了Log4j 1.x，成立了新的项目Log4j 2, Log4j 2具有Logback的所有特性。



**日志框架关系**

> - Log4j 2与Log4j 1发生了很大的变化，Log4j 2不兼容Log4j 1。
> - Commons Logging和Slf4j是日志门面(门面模式是软件工程中常用的一种软件设计模式，也被称为正面模式、外观模式。它为子系统中的一组接口提供一个统一的高层接口，使得子系统更容易使用)。Log4j和Logback则是具体的日志实现方案。可以简单的理解为接口与接口的实现，调用者只需要关注接口而无需关注具体的实现，做到解耦。
> - 比较常用的组合使用方式是Slf4j与Logback组合使用，Commons Logging与Log4j组合使用。
> - Logback必须配合Slf4j使用。由于Logback和Slf4j是同一个作者，其兼容性不言而喻。



## 二、日志框架

### 2.1 Commons Logging

> common-logging是apache提供的一个通用的日志接口，
> 在common-logging中，有一个Simple logger的简单实现，但是它功能很弱，所以使用common-logging，通常都是配合着log4j来使用；
> Commons Logging定义了一个自己的接口 org.apache.commons.logging.Log，以屏蔽不同日志框架的API差异，这里用到了Adapter Pattern（适配器模式）。

### 2.2 SLF4J

> Simple Logging Facade for Java（SLF4J）用作各种日志框架（例如java.util.logging，logback，log4j）的简单外观或抽象，允许最终用户在部署时插入所需的日志框架。
> SLF4J不依赖于任何特殊的类装载机制。 实际上，每个SLF4J绑定在编译时都是硬连线的，以使用一个且只有一个特定的日志记录框架。 例如，slf4j-log4j12-1.8.0-beta2.jar绑定在编译时绑定以使用log4j。 在您的代码中，除了slf4j-api-1.8.0-beta2.jar之外，您只需将您选择的一个且只有一个绑定放到相应的类路径位置。 不要在类路径上放置多个绑定。
> 因此，slf4j 就是众多日志接口的集合，他不负责具体的日志实现，只在编译时负责寻找合适的日志系统进行绑定。具体有哪些接口，全部都定义在slf4j-api中。查看slf4j-api源码就可以发现，里面除了public final class LoggerFactory类之外，都是接口定义。因此，slf4j-api本质就是一个接口定义。

### 2.3 Log4j

> Apache Log4j是一个非常古老的日志框架，并且是多年来最受欢迎的日志框架。 它引入了现代日志框架仍在使用的基本概念，如分层日志级别和记录器。
> 2015年8月5日，该项目管理委员会宣布Log4j 1.x已达到使用寿命。 建议用户使用Log4j 1升级到Apache Log4j 2。

### 2.4 Log4j2

> Apache Log4j 2是对Log4j的升级，它比其前身Log4j 1.x提供了重大改进，并提供了Logback中可用的许多改进，同时修复了Logback架构中的一些固有问题。
> 与Logback一样，Log4j2提供对SLF4J的支持，自动重新加载日志配置，并支持高级过滤选项。 除了这些功能外，它还允许基于lambda表达式对日志语句进行延迟评估，为低延迟系统提供异步记录器，并提供无垃圾模式以避免由垃圾收集器操作引起的任何延迟。
> 所有这些功能使Log4j2成为这三个日志框架中最先进和最快的。

### 2.5 Logback

> ogback是由log4j创始人设计的又一个开源日志组件，作为流行的log4j项目的后续版本，从而替代log4j。
> Logback的体系结构足够通用，以便在不同情况下应用。 目前，logback分为三个模块：logback-core，logback-classic和logback-access。
>
> > logback-core：模块为其他两个模块的基础。
> > logback-classic：模块可以被看做是log4j的改进版本。此外，logback-classic本身实现了SLF4J API，因此可以在logback和其他日志框架（如log4j或java.util.logging（JUL））之间来回切换。
> > logback-access：模块与Servlet容器（如Tomcat和Jetty）集成，以提供HTTP访问日志功能。



## 三、扩展知识点

### 3.1 Commons Logging与Slf4j实现机制对比

Commons Logging实现机制

> Commons Logging是通过动态查找机制，在程序运行时，使用自己的ClassLoader寻找和载入本地具体的实现。详细策略可以查看commons-logging-*.jar包中的org.apache.commons.logging.impl.LogFactoryImpl.java文件。由于Osgi不同的插件使用独立的ClassLoader，Osgi的这种机制保证了插件互相独立, 其机制限制了Commons Logging在Osgi中的正常使用。

Slf4j实现机制

> Slf4j在编译期间，静态绑定本地的Log库，因此可以在Osgi中正常使用。它是通过查找类路径下org.slf4j.impl.StaticLoggerBinder，然后在StaticLoggerBinder中进行绑定。

### 3.2 Slf4j用法

> Slf4j的设计思想比较简洁，使用了Facade设计模式，Slf4j本身只提供了一个slf4j-api-version.jar包，这个jar中主要是日志的抽象接口，jar中本身并没有对抽象出来的接口做实现。
> 对于不同的日志实现方案(例如Logback，Log4j...)，封装出不同的桥接组件(例如logback-classic-version.jar，slf4j-log4j12-version.jar)，这样使用过程中可以灵活的选取自己项目里的日志实现。

**调用关系**

![slf4j-bind](http://img.hurenjieee.com/uPic/slf4j-bind.png)



**日志接入方式**

| jar包名                                                      | 说明                                                         |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| **slf4j-log4j12-1.7.13.jar**                                 | Log4j1.2版本的桥接器，你需要将Log4j.jar加入Classpath。       |
| **slf4j-jdk14-1.7.13.jar**                                   | java.util.logging的桥接器，Jdk原生日志框架。                 |
| **slf4j-nop-1.7.13.jar**                                     | NOP桥接器，默默丢弃一切日志。                                |
| **slf4j-simple-1.7.13.jar**                                  | 一个简单实现的桥接器，该实现输出所有事件到System.err. 只有Info以及高于该级别的消息被打印，在小型应用中它也许是有用的。 |
| **slf4j-jcl-1.7.13.jar**                                     | Jakarta Commons Logging 的桥接器. 这个桥接器将Slf4j所有日志委派给Jcl。 |
| **logback-classic-1.0.13.jar(requires logback-core-1.0.13.jar)** | Slf4j的原生实现，Logback直接实现了Slf4j的接口，因此使用Slf4j与Logback的结合使用也意味更小的内存与计算开销 |

![image](http://img.hurenjieee.com/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2NDcwMzU=,size_16,color_FFFFFF,t_70-20211221182630912-20211222113656898.png)



### 3.3 slf4j-api中几个核心类与接口

| 类与接口                                 | 用途                                                         |
| :--------------------------------------- | :----------------------------------------------------------- |
| org.slf4j.LoggerFactory(class)           | 给调用方提供的创建Logger的工厂类，在编译时绑定具体的日志实现组件 |
| org.slf4j.Logger(interface)              | 给调用方提供的日志记录抽象方法，例如debug(String msg),info(String msg)等方法 |
| org.slf4j.ILoggerFactory(interface)      | 获取的Logger的工厂接口，具体的日志组件实现此接口             |
| org.slf4j.helpers.NOPLogger(class)       | 对org.slf4j.Logger接口的一个没有任何操作的实现，也是Slf4j的默认日志实现 |
| org.slf4j.impl.StaticLoggerBinder(class) | 与具体的日志实现组件实现的桥接类，具体的日志实现组件需要定义org.slf4j.impl包，并在org.slf4j.impl包下提供此类，注意在slf4j-api-version.jar中不存在org.slf4j.impl.StaticLoggerBinder，在源码包slf4j-api-version-source.jar中才存在此类 |

>  参考地址：
>
>  Java 常识（002）：常用日志框架（log4j，slf4j，logback）有啥区别：https://blog.csdn.net/u010647035/article/details/85037206
>
>  Java常用日志框架介绍：https://www.cnblogs.com/chenhongliang/p/5312517.html