搭建Zeppelin本地开发环境

# 简介 

Zeppelin项目主要的编程语言是Java，所以很适合在Intellij里搭建开发环境。结合文章将在本地搭建Zeppelin开发环境，并且在Intellij里Debug。

# Build Zeppelin

Zeppelin本身有很多模块，如果你要全部build的话可能会花费很长的时间（取决于你的网络情况）。但是一般情况下你只使用一部分interpreter，所以只需要build一部分模块就可以了。Zeppelin里的模块主要分为3大类

- Core 模块
- Plugin 模块
- Interpreter 模块

## Core模块

Core模块是Zeppelin的必不可少的核心模块，主要有zeppelin-interpreter，zeppelin-zengine，zeppelin-server，zeppelin-web，你可以运行以下命令来build核心模块。因为其他模块都依赖这些core模块，所以要用install，而不是package

```
mvn clean install -DskipTests -DskipRat -pl zeppelin-server,zeppelin-web -am -Phadoop2
```

## Plugin模块

Zeppelin有几类茶几爱你在zeppelin-plugin目录，目前有2大类：

* notebookrepo
* Launcher

![image-20220506150053295](http://img.hurenjieee.com/uPic/image-20220506150053295.png)

一般情况下是不需要build插件的，但是如果需要一些特殊的查插件功能，就需要编辑zeppelin-plugins模块。比如需要使用HDFS来存储Notebook（使用filesystem），或者需要使用Flinck Interpreter（依赖flink）。

运行下面的命令来编译 plugin 模块（注意Plugin模块依赖Core模块，所以先确保你已经安装了Core模块）

```
cd zeppelin-plugins
mvn clean package -DskipTests
```

## Interpreter模块

Interpreter 模块是指 Zeppelin支持的所有interpreter，这些模块的编译是可选的，取决于你是否使用这个模块。
一般的interpreter模块只要cd到 interpreter 模块文件夹(比如shell，markdown，jdbc），然后运行下面的命令就可以：

```
mvn clean package -DskipTests
```

但是有些interpreter 比较特殊，比如 Spark interpreter 还依赖 Python interpreter 和 R interpreter，本身又有多个子模块。下面例举2个比较特殊也是常用的Interpreter 模块。

### **编译 Spark interpreter**

master branch

```
mvn clean package -DskipTests -pl spark/interpreter,spark/scala-2.11,spark/scala-2.12,spark/scala-2.13 -am
```

0.10.1 以及之前版本

```
mvn clean package -DskipTests -pl spark/interpreter,spark/scala-2.11,spark/scala-2.12 -am
```

### **编译 Flink interpreter**

```
mvn clean package -DskipTests -pl flink/flink-scala-2.11,flink/flink-scala-2.12 -am
```



# **Debug Zeppelin**

Zeppelin有两类进程：

- Zeppelin Server 进程
- Interpreter 进程

这两类进程的Debug 方式会略有不同



## Debug Zeppelin Server 进程

**IDEA内部DEBUG**

1. Build 了 Zeppelin 的 Core 模块。

2. 找到 Zeppelin Server 的入口class：org.apache.zeppelin.server.ZeppelinServer

3. 设置以下两个环境变量：

   1. ZEPPELIN_HOME  设置为你的zeppelin源码目录，这样conf目录下的zeppelin-site.xml 才会被正确加载。
   2. ZEPPELIN_LOCAL_IP：设置为127.0.0.1，这个是为了保证zeppelin server进程和interpreter 进程的通信。这是因为在某些特殊情况下(比如你开了VPN），zeppelin server 和 interpreter 进程会因为没有找对ip而导致通信失败，所以需要这个环境变量。

4. 开启DEBUG

   ![image-20220506152825646](http://img.hurenjieee.com/uPic/image-20220506152825646.png)

   接下来就可以设置断点，Debug Zeppelin Server进程了。
   Zeppelin Server进程只依赖Core模块，如果你更改了Core代码，无需重新Build Core模块，只要重启Zeppelin Server进程就可以。

## Debug Interpreter 进程

**远程DEBUG**

1. Build了对应的interpreter 模块

2. 修改zeppelin-env.sh的 ZEPPELIN_INTP_JAVA_OPTS，开启远程DEBUG。

   > ```
   > export ZEPPELIN_INTP_JAVA_OPTS="-verbose:class -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,address=6006"
   > ```

3. 在Intellij里设置Remote Debug, 端口就是Step 2设置的6006

   > -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=6002

   ![image-20220506152951266](http://img.hurenjieee.com/uPic/image-20220506152951266.png)

4. 在 Zeppelin 页面运行一个段落，启动某个interpreter进程。
   这时候你会发现段落状态一直处于PENDING状态，这是因为Step 2设置了ZEPPELIN_INTP_JAVA_OPTS的缘故，导致interpreter 进程一直没有起来，这时候你要启动Step 3的Remote debug，这样你的interpreter 进程就进入debug模式，你就可以设置断点来debug了。
   如果你修改了Interpreter代码，需要重新Build interpreter 模块，然后重启Interpreter 进程。

# 其他问题

## IDEA Maven问题

> 问题：idea上直接对maven intall存在问题，如果直接install，提示部分依赖包（Zeppelin: Zengine）不存在。

> 解决：直接点击的话，没有-am参数，直接去maven仓库直接查找，项目刚clone下来的时候，没有全局install的话，工程内的包没有install，那肯定会缺少依赖的。

## IDEA 全局install

> 问题：idea在root上使用install命令时可能出现一下提示
>
> > [ERROR] Failed to execute goal org.apache.rat:apache-rat-plugin:0.13:check (verify.rat) on project zeppelin: Too many files with unapproved license: 1 See RAT report in: /Users/jack/Work/Github/zeppelin/target/rat.txt -> [Help 1]
> > [ERROR] 
> > [ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
> > [ERROR] Re-run Maven using the -X switch to enable full debug logging.
> > [ERROR] 
> > [ERROR] For more information about the errors and possible solutions, please read the following articles:
> > [ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException

> 解决：文件License信息检查不通过。
> 增加-Drat.skip=true跳过License检查。

## zeppelin-web:npm资源问题

> 问题：npm前端资源长时间无法install
>
> ![image-20220506192155648](http://img.hurenjieee.com/uPic/image-20220506192155648.png)
>
> 解决：问题是因为npm仓库无法访问，可以和官网一样设置代理。同样也可以使用国内镜像仓库。但是，`--registry=https://registry.npm.taobao.org`并不可用，因为地址已经写死了，在package-lock.json文件中，替换成淘宝地址即可。





> ### 参考地址
>
> [如何搭建Zeppelin本地开发环境](https://mp.weixin.qq.com/s/FrZ9NVxzaKqtXXsaRcxnAQ)