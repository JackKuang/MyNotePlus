# Oracle连接问题

## 一、起因

原先公司的Kettle运行模式为在Windows上界面操作，很大程度上依赖于Windows上Kettle主程序。一旦Kettle任务主程序挂掉了，结果就导致了任务开发简单，但是运维Kettle任务麻烦。

于是，进行了任务迁移，运行环境迁移到Linux，使用kettle的kitchen进行调度执行。同时，增加了调度平台，做统一的监控和调度。

整体架构而言，会有一个调度平台来调度执行任务（XXL-JOB、DolphinScheduler、Azkaban等等均可），任务的执行节点会提前部署好Kettle的运行环境，直接运行即可。

但是，遇到了一个问题，在一段时间内，任务会有异常，且Oracle连接会有异常：

![image-20201222110624939](http://img.hurenjieee.com/uPic/image-20201222110624939.png)

然而，这个问题，在Windows上没有看到过。



##  二、处理

### 2.1 网络原因

一开始，看到这个问题，最先考虑的是数据库或者网络问题，但是通过运维等方面了解到，实际数据库都是正常的。那就开始考虑网络问题，于是写了一段脚本，并使用crontab定时调度。

```sh
#/bin/bash
echo "================" >> test.log 
date >> test.log
ping host -c 1 >> test.log
telnet host >> test.log
```

但是实际排查之后，网络端口都是通的，kettle任务还是连接异常。



### 2.2 其他原因

网络正常，运行环境正常，就思考了。是什么问题导致的？同样是Java运行环境、同样是kettle任务、连JDBC驱动的jar包也是一样的、甚至连的Oracle也是一样的、网络正常的。一时不知道原因是啥。



### 2.3 Linux + Java + Oracle + 连接异常

问题排查不出来，就上搜索。果然找到了问题来源：

> https://blog.csdn.net/u011429743/article/details/108278807

oracle JDBC在建立连接时需要一些随机数据用以加密session token之类的东西，而这个随机数据源默认用的是/dev/random。这个随机数生成器可能会导致随机数生成很慢。

在启动Java程序中加入配置：

```
-Djava.security.egd=file:///dev/urandom
```

在java环境中配置了这个选项之后，后面的任务都正常运行了。



## 三、思考

/dev/random和/dev/urandom是Linux系统中提供的随机伪设备，这两个设备的任务，是提供永不为空的随机字节数据流。很多解密程序与安全应用程序（如SSH Keys,SSL Keys等）需要它们提供的随机数据流。

这两个设备的差异在于：/dev/random的random pool依赖于系统中断，因此在系统的中断数不足时，/dev/random设备会一直封锁，尝试读取的进程就会进入等待状态，直到系统的中断数充分够用, /dev/random设备可以保证数据的随机性。/dev/urandom不依赖系统的中断，也就不会造成进程忙等待，但是数据的随机性也不高。