# SpringCloud

## 一、SpringCloud入门

### 1.1 Spring介绍

* 微服务的概念源于2014年3月Martin Fowler所写的一篇文章“Microservices”。
* 微服务架构是一种架构模式，它提倡将单一应用程序划分成一组小的服务，服务之间互相协调、互相配合，为用户提供最终价值。每个服务运行在其独立的进程中，服务与服务间采用轻量级的通信机制互相沟通（通常是基于HTTP的RESTful API）。每个服务都围绕着具体业务进行构建，并且能够被独立地部署到生产环境、类生产环境等。另外，应尽量避免统一的、集中式的服务管理机制，对具体的一个服务而言，应根据业务上下文，选择合适的语言、工具对其进行构建。
* 微服务是一种架构风格，一个大型复杂软件应用由一个或多个微服务组成。系统中的各个微服务可被独立部署，各个微服务之间是松耦合的。每个微服务仅关注于完成一件任务并很好地完成该任务。在所有情况下，每个任务代表着一个小的业务能力。

### 1.2 Spring优势

* **复杂度可控**

  > 在将应用分解的同时，规避了原本复杂度无止境的积累。每一个微服务专注于单一功能，并通过定义良好的接口清晰表述服务边界。由于体积小、复杂度低，每个微服务可由一个小规模开发团队完全掌控，易于保持高可维护性和开发效率。

* 独立部署

  > 由于微服务具备独立的运行进程，所以每个微服务也可以独立部署。当某个微服务发生变更时无需编译、部署整个应用。由微服务组成的应用相当于具备一系列可并行的发布流程，使得发布更加高效，同时降低对生产环境所造成的风险，最终缩短应用交付周期。

* 技术选型灵活

  > 微服务架构下，技术选型是去中心化的。每个团队可以根据自身服务的需求和行业发展的现状，自由选择最适合的技术栈。由于每个微服务相对简单，故需要对技术栈进行升级时所面临的风险就较低，甚至完全重构一个微服务也是可行的。

* 容错

  > 当某一组建发生故障时，在单一进程的传统架构下，故障很有可能在进程内扩散，形成应用全局性的不可用。在微服务架构下，故障会被隔离在单个服务中。若设计良好，其他服务可通过重试、平稳退化等机制实现应用层面的容错。

* 扩展

  > 单块架构应用也可以实现横向扩展，就是将整个应用完整的复制到不同的节点。当应用的不同组件在扩展需求上存在差异时，微服务架构便体现出其灵活性，因为每个服务可以根据实际需求独立进行扩展。

## 二、SpringCloud技术栈

![image-20201113172645656](http://img.hurenjieee.com/uPic/image-20201113172645656.png)

![image-20201113172747871](http://img.hurenjieee.com/uPic/image-20201113172747871.png)

* 2020 H版内容

  ![image-20201113172929396](http://img.hurenjieee.com/uPic/image-20201113172929396.png)

* Cloud 升级

  * 服务注册中心

    > Eureka：❌
    >
    > Zookeeper：✅
    >
    > Consul：✅
    >
    > Nacos：🌟阿里巴巴

  * 服务调用

    > Ribbon：✅
    >
    > LoadBalancer：新技术

  * 服务调用2

    > Feign：❌
    >
    >  OpenFeign：✅

  * 服务降级

    > Hystrix：❌
    >
    > Resilience4j：✅
    >
    > Sentienl：✅推荐

  * 服务网关

    > Zuul：❌
    >
    > Zuul2：❌
    >
    > Gateway：✅

  * 服务配置

    > Config：❌
    >
    > Nacos：🌟

  * 服务总线

    > Bus：❌
    >
    > Necos：✅

英文文档

中文文档：https://www.bookstack.cn/read/spring-cloud-docs/docs-index.md

## 三、Spring Boot 和 Spring Cloud

* SpringBoot必须要提升到2.0版本以上

* Spring Cloud采用了英国伦敦地铁站的名称来命名，并由地铁站名称字母A Z依次类推的形式来发布迭代版本。

  >  SpringCloud是一个由许多 子项目组成的综合项目，各子项目有不同的发布节奏。为了管理SpringCloud与各子项目的版本依赖关系，发布了一个清单，其中包括了某个SpringCloud版本对应的子项目版本。
  >
  > 为了避免SpringCloud版本号与子项目版本号混淆， SpringCloud版本采用 了名称而非版本号的命名，这些版本的名字采用了伦敦地铁站的名字，根据字母表的顺序来对应版本时间顺序。
  >
  > 例如Angel是第一个版本， Brixton是第二个版本。
  >
  > 当SpringCloud的发布内容积累到临界点或者一个重大BUG被解决后，会发布一个"service releases "版本，简称SRX版本，比如Greenwich.SR2就是SpringCloud发布的Greenwich版本的第2个SRX版本。

* Spring Cloud和Spring Boot版本对比

  | Release Train                                                | Boot Version                     |
  | :----------------------------------------------------------- | :------------------------------- |
  | [Hoxton](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Hoxton-Release-Notes) | 2.2.x, 2.3.x (Starting with SR5) |
  | [Greenwich](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Greenwich-Release-Notes) | 2.1.x                            |
  | [Finchley](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Finchley-Release-Notes) | 2.0.x                            |
  | [Edgware](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Edgware-Release-Notes) | 1.5.x                            |
  | [Dalston](https://github.com/spring-projects/spring-cloud/wiki/Spring-Cloud-Dalston-Release-Notes) | 1.5.x                            |

  配置要求地址：https://start.spring.io/actuator/info
  
  

## 四、Eureka 服务注册与发现

### 4.1 简介

Spring Cloud Euraka是Spring Cloud集合中的一个组件，它是对Euraka的集成，用于服务注册与发现。

Eureka采用了CS的设计架构，Eureka Server作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用Eureka的客户端连接到Eureka Server并维持心跳连接。这样系统的维护人员就可以通过Eureka Server来监控系统中各个微服务是否正常运行。

![在这里插入图片描述](http://img.hurenjieee.com/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3F3ZTg2MzE0,size_16,color_FFFFFF,t_70.png)



* Eureka Server提供服务注册服务

  > 各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

* Eureka Client通过注册中心进行访问

  > Client是一个Java客户端，用于简化Eureka Server的交互，客ru户端同时也具备了一个内置、使用轮训(round-robin)算法负载均衡器。在应用启动后，将会向Eureka Server发送心跳（默认周期为30秒）。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒）。

### 4.2 自我保护机制

* 为什么会产生Eureka自我保护机制

> EurekaClient可以正常运行，但是与EurekaServer网络不通的情况下。Eureka不会立即将EurekaClient服务删除。

* 什么是自我保护模式

> 默认情况下，如果EurekaServer在一定时间内没有接受到某个微服务实例的心跳，EurekaServer将会注销该实例（默认90s）。但是当网络分区故障发生（延时、卡顿、拥挤）时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本省是健康的，**此时不应该注销这个微服务。**Eureka通过"自我保护模式"来解决问题——当EurekaServer节点在短时间内丢失了过多的客户端时（可能发生网络分区故障），那么节点就会进入自我保护模式。

 

自我保护机制：默认情况EurekaClient定时向EurekaServer端发送心跳包

如果Eureka在Server端一定时间内（默认90秒）没有收到EurekaClient发送心跳包，便会注解从服务注册列表中剔除该服务，但是在短时间内丢失大量的服务心跳，这时候EurekaServer会开启自我保护机制，不会剔除该服务（该现象可能出现子啊如果网络不通，但是EurekaClient未出现宕机，此时如果换做别的注册中心如果一定时间没有收到心跳将剔除该服务，这样就出现了严重失误，因为客户端还能发送正常心跳，只是网络延迟问题，而保护机制是为了解决此问题而产生的）。

在自我保护模式中，Eureka Server会保护服务注册中的信息，不在注销任务服务实例。

它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任务可能健康的服务实例。==>**好死不如烂活着。**



综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留）也不盲目注销任何健康的微服务。使用自我模式，可以让Eureka集群更加健壮、稳定。

### 4.3 Eureka停更

https://github.com/Netflix/eureka/wiki

## 五、Zookeeper

### 5.1 介绍



## 六、Consul

### 6.1介绍

Consul是一套开源的分布式服务发现和配置管理系统，由GO语言开发。

官网：https://www.consul.io/

Consul提供以下功能



![image-20210807160245929](http://img.hurenjieee.com/uPic/image-20210807160245929.png)

* 服务发现
* 健康检查
* KV健值对存储
* 安全加固
* 多数据中心



### 6.2 安装

Docker安装

```sh
docker pull consul
docker rm -f consul
docker run -d --name consul -p 8500:8500 -e CONSUL_BIND_INTERFACE=eth0 consul
```

### 6.3 三者对比

![](http://img.hurenjieee.com/uPic/640.jpg)

![image-20210809222348741](http://img.hurenjieee.com/uPic/image-20210809222348741.png)



**CAP**

> 一个分布式系统不可能同时满足一致性，可用性和分区容错性这个三个需求，因此，根据CAP原理将NoSql分成满足CA原则、满足CP原则和满足AP原则三大类：
>
> * CA—单点集群，满足一致性、可用性的系统，通常在可扩展性上不太强大。
> * CP—满足一致性，分区容错性的系统，通常性能不是特别高。
> * AP—满足可用性，分区容错性的系统，通常啃根对一致性要求低一点。



## 七、Ribbon

### 7.1 介绍

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。主要功能是**提供客户端的软件负载均衡算法和服务调用。**

Ribbon客户端组件提供一系列完善的配置项如连接超时，重拾。简单的说，就是配置文件中列出Load Balancer后面所有的机器，Ribbon会自动帮助你基于某种规则（如简单轮训、随机连接等）去连接这些机器。 

Wiki：https://github.com/Netflix/ribbon/wiki



### 7.2 核心组件IRule

![image-20210811202212047](http://img.hurenjieee.com/uPic/image-20210811202212047.png)

常用的IRule组件：

* RoundRobinRule——轮训
* RandomRule——随机
* RetryRule——重试
* WeightedResponseTimeRule——RoundRobinRule的扩展，速度越快权重越大
* BestAvailableRule——过滤故障，最稳定
* AvailabilityFilteringRule——过滤故障，并发较小
* ZoneAvoidancePredicate——复合判断Server所在区域的性能和Server的可用性



## 八、OpenFeign

### 8.1 介绍

Fiegn是一个声明式的Web服务客户端，让编写Web服务器客户端变得非常容易。 只需要创建一个接口并添加注解就可以了。

Feign旨在使编写Java Http客户端变得更容易。

前面在使用Ribbon+RestTemplate时，利用RestTemplate对http请求的封住处理，形成了一套模版化的调用方法。但是在实际开发中，由于对依赖服务的调用可能不止一处，**往往一个接口被多处调用，所以通常对都会针对每个微服务自行封装一些客户端类来包装，我们只需要创建一个借口并且使用注解的方式来配置它**，即可完成对服务提供方的接口绑定，简化了使用Spring Cloud Ribbon时，自动封装服务调用客户端的开发量。

利用Ribbon维护了服务提供方的服务列表信息，并且通过轮训实现了客户端的负载均衡。而与Ribbon不同的是，**通过Feign只需要定义服务绑定接口且以声明式的方法**，优雅而简单的实现了服务调用。



## 九、Hystrix

### 9.1 介绍

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），**向调用方返回一个符合预期的、可处理的备选响应，而不是长时间的等待或者调用方无法处理的异常**，这样就保证了服务调用方的线程不会被长时间吗、不必要地占用，从而避免故障在分布式系统中的蔓延，乃至雪崩。



### 9.2 重要概念

#### 9.2.1 服务降级-fallback

**服务器忙，清稍后重试，不让客户端等待并立刻返回一个友好提示。**

触发的原因：

* 程序运行异常
* 超时
* 服务熔断触发服务降级
* 线程池打满了，资源不足

```java

@HystrixCommand(fallbackMethod = "timeoutHandler", commandProperties = {
  @HystrixProperty(name = HystrixPropertiesManager.EXECUTION_ISOLATION_THREAD_TIMEOUT_IN_MILLISECONDS, value = "3000")
})
public String paymentTimeout(Long id) {
}
```

#### 9.2.2 服务熔断-break

**达到最大服务访问后，直接拒绝访问，然后调用服务降级的方法返回友好提示。**

```java

@HystrixCommand(fallbackMethod = "paymentCircuitBreakerHandler", commandProperties = {
@HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_ENABLED, value = "true"),
@HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_REQUEST_VOLUME_THRESHOLD, value = "10"),
@HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_SLEEP_WINDOW_IN_MILLISECONDS, value = "10000"),
@HystrixProperty(name = HystrixPropertiesManager.CIRCUIT_BREAKER_ERROR_THRESHOLD_PERCENTAGE, value = "60"),
})
public String paymentCircuitBreaker(Long id) {
```



熔断器模式的调用流程:

<img src="http://img.hurenjieee.com/uPic/v2-9726fd94f8403ac147178e2086306853_b.png" alt="img" style="zoom:50%;" />

从图中可以看出，当超时出现的次数达到一定条件后，熔断器会触发打开状态，客户端的下次调用将会直接返回，不用等待超时产生。

在熔断器内部，往往有以下几种状态：

![img](http://img.hurenjieee.com/uPic/v2-f9b85f1cd0db655e108e89a31edffc18_b.png)

1. **闭合（Closed）状态**：该状态下能够对目标服务或者方法进行正常的调用。熔断器类维护了一个时间窗口内调用失败的次数，如果某次调用失败，则失败次数加1。如果最近失败次数超过了在给定的时间窗口内允许失败的阈值（可以是数量也可以是比例），则熔断器类切换到断开（Open）状态。此时熔断器设置了一个计时器，当时钟超过了该时间，则切换到半断开（Half-Open）状态，该睡眠时间的设定是给了系统一次机会来修正导致调用失败的错误。
2. **断开（Open）状态**：该状态下，对目标服务或方法的请求会立即返回错误响应，如果设置了fallback方法，则会进入fallback的流程。
3. **半断开（Half-Open）状态**：允许对目标服务或方法的一定数据的请求可以去调用服务。如果这些请求服务的调用成功，那么可以认为之前导致失败的错误已经修正，此时熔断器切换到闭（并且将错误计数器重置）；如果这一定数据的请求有调用失败的情况，则认为导致之前调用失败的问题仍然存在，熔断器切回到断开方式，然后开启重置计时器来给系统一定的时间来修正错误。半断开状态能够有效防止正在恢复中的服务被突然而来的大量请求再次拖垮。

### 9.2.3 服务限流-flowlimit

秒杀高并发操作，严谨一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行。





### 9.3 流程图

https://github.com/Netflix/Hystrix/wiki/How-it-Works

![img](http://img.hurenjieee.com/uPic/hystrix-command-flow-chart-640.png)

1. [Construct a `HystrixCommand` or `HystrixObservableCommand` Object](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow1)
2. [Execute the Command](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow2)
3. [Is the Response Cached?](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow3)
4. [Is the Circuit Open?](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow4)
5. [Is the Thread Pool/Queue/Semaphore Full?](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow5)
6. [`HystrixObservableCommand.construct()` or `HystrixCommand.run()`](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow6)
7. [Calculate Circuit Health](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow7)
8. [Get the Fallback](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow8)
9. [Return the Successful Response](https://github.com/Netflix/Hystrix/wiki/How-it-Works#flow9)



### 9.4 HystrixDashboard

除了隔离依赖服务的调用以外，Hystrix还提供了**准实时的调用监控（Hystrix Dashboard）**，Hystrix回持续地记录所有通过Hystrix发起的请求的执行信息，并以报表和图形的形式展示给用户，包括每秒执行多少请求、多少成功、多少失败等。Netflix通过hystrix-metrics-event-stream项目实现了对以上指标的监控。Spring Cloud也提供了Hystrix Dashboard的整合，对监控内容转化成可视化界面。

![image-20211027113535703](../../../../../../Library/Application%20Support/typora-user-images/image-20211027113535703.png)

被监听服务地址如果404，则需要增加以下配置：

```java

    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
```

![image-20211027113810444](http://img.hurenjieee.com/uPic/image-20211027113810444.png)

