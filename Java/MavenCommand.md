> 使用maven很长一段时间了，但是用的场景大部分是管理jar包，很多用法都没有详细用到。最近打包zeppelin的时候看到命令的很多参数，特地再系统性学习一下maven。
>
> 

# 一、介绍

Maven是基于项目对象模型(POM project object model)。
可以通过一小段描述信息（配置）来管理项目的构建，报告和文档的软件项目管理工具。
简单的说就是用来管理项目所需要的依赖且管理项目构建的工具。

作用：

* 统一开发规范与工具
* 统一管理jar包



# 二、Maven的生命周期

Maven把项目的构建划分为不同生命周期（lifecycle）：

1. clean：清理
2. validate：验证
3. complie： 编译
4. test：测试
5. pacakge：打包(包含complie、test)
6. verify：检查。对集成测试的结果进行检查，以保证质量达标。
7. install：pageage之后部署到本地maven仓库
8. site：站点部署
9. deploy：pageage之后部署到本地maven仓库和远程仓库



# 三、Maven规范

maven使用如下几个要素来唯一定位当前运行项目的输出：

* groupId：公司、机构、团体
* artifactId：项目id
* version：版本
  * SNAPSHOT：快照包，还在开发过程中，变更频繁。
  * LASTEST：最新发布包
  * RELEASE：最后一个发布版
* packaging：打包输出类型，默认jar，也可以是war包。



# 四、Maven项目依赖

## 4.1 多模块依赖和继承

目录结构：

> parent
>     ├─childA(model层)
>     │  └─pom.xml(jar)
>     ├─childB(web层) 
>     │  └─pom.xml(war)  
>    └─pom.xml(pom) 

parent的pom需要引入module

>  <modules>
>     <module>childA</module>   <!-- 不加module则不会被联合编译 -->
>     <module>childB</module>
>  </modules>

**parent中执行mvn install就能将 childA和childB 一起编译**

childA和childB的pom.xml都需要配置parent，防止引入的包冲突(如果不加parent，会分别去编译他们引入的依赖，会重复引入包)。



## 4.2 子项目继承父项目的依赖

parent中加上**dependencyManagement**，child项目就可以继承parent项目的依赖，并且在child中可以不用加version了。



## 4.3 依赖范围

dependency中还有scope属性，表示对依赖包的具体管理：

1. compile：默认配置，将会一起打包
2. provided：运行环境中已经提供了，不需要打包，比方说tomcat的Servlet。
3. runtime：运行和测试时需要，编译时不需要。
4. test：测试时需要
5. system：结合systemPath使用，不建议使用，



## 4.4 可选依赖

A依赖B时，设置为B依赖时可选(option)的。当C依赖A时，默认就不会加载B，如果需要用到B，需要额外配置对B的依赖。



## 4.5 排除依赖

A依赖B时。当C依赖A时，默认就会加载B，如果需要不需要用到B，需要额外排除(exclusions)对B的依赖。



## 4.6 dependency中的type属性

当我们需要引入很多jar包的时候会导致pom.xml过大，我们可以想到的一种解决方案是定义一个父项目，但是父项目只有一个，也有可能导致父项目的pom.xml文件过大。这个时候我们引进来一个type为pom，意味着我们可以将所有的jar包打包成一个pom，然后我们依赖了pom，即可以下载下来所有依赖的jar包。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>${spring.boot.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

# 五、Maven插件机制

|plugin|function|lifecycle phase|
| ---- | ---- | ---- |
|maven-clean-plugin|清理上一次执行创建的目标文件|clean|
|maven-resources-plugin|处理源资源文件和测试资源文件|resources,testResources|
|maven-compiler-plugin|编译源文件和测试源文件|compile,testCompile|
|maven-surefire-plugin|执行测试文件|test|
|maven-jar-plugin|创建 jar|jar|
|maven-install-plugin|安装 jar，将创建生成的 jar 拷贝到 .m2/repository 下面|install|
|maven-deploy-plugin|发布 jar|deploy|



# 六、mvn运行其他参数

```sh
> $ mvn --help                                                                                                                                                                                                                       

usage: mvn [options] [<goal(s)>] [<phase(s)>]

Options:

# 构建指定模块,同时构建指定模块依赖的其他模块;(向下)
 -am,--also-make                        If project list is specified, also
                                        build projects required by the
                                        list
# 构建指定模块,同时构建依赖于指定模块的其他模块;(向上)
-amd,--also-make-dependents            If project list is specified, also
                                        build projects that depend on
                                        projects on the list
# 以批处理(batch)模式运行;
 -B,--batch-mode                        Run in non-interactive (batch)
                                        mode (disables output color)
# 以批处理(batch)模式运行;
 -b,--builder <arg>                     The id of the build strategy to
                                        use
# 检查不通过,则构建失败;(严格检查)
 -C,--strict-checksums                  Fail the build if checksums don't
                                        match
# 检查不通过,则警告;(宽松检查)
 -c,--lax-checksums                     Warn if checksums don't match
 -cpu,--check-plugin-updates            Ineffective, only kept for
                                        backward compatibility
# 定义系统变量
 -D,--define <arg>                      Define a system property
# 显示详细错误信息
 -e,--errors                            Produce execution error messages
# 生成Master password的密文
 -emp,--encrypt-master-password <arg>   Encrypt master security password
# 加密访问服务器的密码
 -ep,--encrypt-password <arg>           Encrypt server password
# 使用指定的POM文件替换当前POM文件
 -f,--file <arg>                        Force the use of an alternate POM
                                        file (or directory with pom.xml)
# 最后失败模式：Maven会在构建最后失败（停止）。如果Maven refactor中一个失败了，Maven会继续构建其它项目，并在构建最后报告失败。
 -fae,--fail-at-end                     Only fail the build afterwards;
                                        allow all non-impacted builds to
                                        continue
# 最快失败模式： 多模块构建时,遇到第一个失败的构建时停止。
 -ff,--fail-fast                        Stop at first failure in
                                        reactorized builds
# 从不失败模式：Maven从来不会为一个失败停止，也不会报告失败。
 -fn,--fail-never                       NEVER fail the build, regardless
                                        of project result
# 替换全局级别settings.xml文件
 -gs,--global-settings <arg>            Alternate path for the global
                                        settings file
# 替换全局级别用户工具链文件
 -gt,--global-toolchains <arg>          Alternate path for the global
                                        toolchains file
# 帮助日志
 -h,--help                              Display help information
# 制定输出日志文件
 -l,--log-file <arg>                    Log file where all build output
                                        will go (disables output color)
# 未知
 -llr,--legacy-local-repository         Use Maven 2 Legacy Local
                                        Repository behaviour, ie no use of
                                        _remote.repositories. Can also be
                                        activated by using
                                        -Dmaven.legacyLocalRepo=true
# 仅构建当前模块，而不构建子模块
 -N,--non-recursive                     Do not recurse into sub-projects
# 【废弃】
 -npr,--no-plugin-registry              Ineffective, only kept for
                                        backward compatibility
# 【废弃】
 -npu,--no-plugin-updates               Ineffective, only kept for
                                        backward compatibility
# 强制不更新SNAPSHOT
 -nsu,--no-snapshot-updates             Suppress SNAPSHOT updates
# 不展示下载和上传进度条
 -ntp,--no-transfer-progress            Do not display transfer progress
                                        when downloading or uploading
# 运行offline模式,不联网进行依赖更新
 -o,--offline                           Work offline
# 激活指定的profile文件列表(用逗号[,]隔开)
 -P,--activate-profiles <arg>           Comma-delimited list of profiles
                                        to activate
# 手动选择需要构建的项目,项目间以逗号分隔;
 -pl,--projects <arg>                   Comma-delimited list of specified
                                        reactor projects to build instead
                                        of all projects. A project can be
                                        specified by [groupId]:artifactId
                                        or by its relative path
# 安静模式,只输出ERROR
 -q,--quiet                             Quiet output - only show errors
# 从指定的项目(或模块)开始继续构建
 -rf,--resume-from <arg>                Resume reactor from specified
                                        project
# 替换用户级别settings.xml文件
 -s,--settings <arg>                    Alternate path for the user
                                        settings file
# 替换用户工具链文件
 -t,--toolchains <arg>                  Alternate path for the user
                                        toolchains file
# 线程统计
 -T,--threads <arg>                     Thread count, for instance 2.0C
                                        where C is core multiplied
# 强制更新releases、snapshots类型的插件或依赖（默认snapshot一天只更新一次）
 -U,--update-snapshots                  Forces a check for missing
                                        releases and updated snapshots on
                                        remote repositories
# 【废弃】
 -up,--update-plugins                   Ineffective, only kept for
                                        backward compatibility
# 版本信息
 -v,--version                           Display version information
# 版本信息，同时展示项目信息
 -V,--show-version                      Display version information
                                        WITHOUT stopping build
# debug模式，输出详细信息
 -X,--debug                             Produce execution debug output
```

总结一下好用的吧

| 参数 | 作用                                    |
| ---- | --------------------------------------- |
| -am  | 构建时同时构建依赖模块                  |
| -D   | 定义系统变量                            |
| -pl  | 手动选择需要构建的项目,项目间以逗号分隔 |
| -U   | 强制更新可能更新的依赖                  |

