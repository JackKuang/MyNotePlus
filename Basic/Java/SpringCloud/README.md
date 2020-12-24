## SpringCloud

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
  
  

## 四、


