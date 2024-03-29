# 复杂场景下的开发

最近在参与公司的迭代项目中，愈发感觉整理开发、架构上的问题，导致代码越来越难维护，也伴随着项目迭代上的风险。于是就有了开发上的一些思考。
导致这个问题的原因有啥：

* 产品设计迭代了，在功能上发生很大的变动。

  > 产品功能设计之初的满足需求并不多，但是随着版本迭代，功能不断叠加，导致很多原先上的功能开发无法支撑需求。
  > 拿一个经历过的例子，产品在起步阶段想法比较简单，产品思路简单，开发也可以很容易支持，但是随着产品不断迭代变化，导致当前的开发技术无法满足，可能导致架构上的调整。而在整体开发计划上，总是会最大化避免以往业务功能上开发。只能以牺牲代码质量来保障业务支撑。

* 开发上处理过于简单，很多地方使用if...else处理。

  > 随着业务发展，试图用if...else解决第一个问题，于是，第二个、第三个都是用if...else来解决。随着if...else越来越多，逻辑在迭代过程中变得越来越乱，最后彻底变成一个看不懂改不动的黑盒子，没有人能搞清楚黑盒子里面到底发生了什么。

* 开发同学比较多，部分接口重复开发。

  > 业务开发还是面向数据库开发，没有抽象业务，导致不同同学开发功能时，按自己的逻辑查询一边数据库，很容易出现重复开发。缺乏团队合作内容。
  
* 测试质量把控

  > 研发团队的同学在功能开发完成之后，缺乏对内容的质量把控，认为产品的bug没有测试出来是测试的问题。

以及其他等等的一些原因，所以在开发整体功能的时候，需要不仅仅是去思考功能实现，更要想到后续功能迭代。

## 一、质量把控

越是复杂系统，其质量就越需要质量把控。这里的质量就是交付质量，但是交付之俩个也可以细分多个层面：

* 系统质量：系统的功能完备性、稳定性和健壮性。
* 产品体验：客户使用产品时的体验。

在大多数时候，质量很容易被理解为测试不足。希望在实际测试、使用中才能考虑到的问题，由问题来驱动质量提升。

由问题驱动的质量提升虽然是一种想法，但是仔细思考还是有很多局限性：

* 不断的BUG测出，然后打补丁做的来的系统质量，质量很难把控。（持续的补丁只会降低系统的可维护性）
* 设计是为了实现功能，功能没问题，并不意味着设计是合理的。（一大堆if else的代码只会降低代码的可维护性）

所以，对于产品的质量把控需要从产品、技术、测试等不同的角度去做，形成统一的意识形态。



## 二、关注产品未来走向

之前有产品同学和我聊天的时候说，做产品比做开发更难。做产品需要思路，了解用户需要什么，怎么实现，是一个探索的过程，需要不断得迭代才能出一个好的产品，不像开发一样，有明确的目的，直接做就可以了。但是实际上，开发也并不是一路畅通，在完成功能研发的路上，还要考虑技术方案设计、处理各种问题、解决技术壁垒等等的问题。
其实我更想说的是，开发同学在开发的路上，需要适当地考虑到产品的未来走向，为未来的迭代做好准备。避免未来技术上无法支撑产品功能时的系统性重构。



## 三、开发思维-解耦

解藕即减少模块之间的互相干扰，让各个模块的修改变化尽可能小的影响其他模块。
解藕的基本方法就是面向接口开发，也就是说各个模块只和接口发生耦合，要么调用接口，要么实现接口。正因如此，接口本身的设计是程序模块化的重中之重，大型程序中接口设计的优劣直接决定整个系统的优劣。
在一些业务中，存在一定程度上的关联依赖，需要避免跨层级调用。举一个简单的例子，在数据库查询上，A业务依赖B业务，而B业务依赖C业务，当A业务调用B业务的时候， 就会有B、C两张表直接联表查询。而当C的业务变更时，会发现除了修改C的业务之外，还需要修改C之上的B。这显然不符合常理。
所以在开发上，必须要做到每一层功能上的解藕，保证业务上的隔离。



## 四、巧用设计模式、设计原则

相信很多同学在开发过程中都会遇到这样的场景：这段代码现在有点小问题，需要fix，最好的处理方案是把功能再抽取出来，但是比较麻烦，而使用if来判断一下，几行代码就可以完成了。很多人选择了使用if，这里也有很多原因：减少代码变动以保障业务、业务不熟怕修改影响比较大。我也见过一大堆代码里面使用了大量的if/switch语句，还出现了相似代码。像这些种种的问题，不断地积累使系统成为了一个不可见的盲盒。
设计模式、设计原则的出现就是能加系统的健壮性，易修改性和可扩展性。更深层次的提供系统的稳定性。所以在写代码的时候，除了思考业务本身之外，还需要思考如何让这段代码看起来是个巧妙、优雅的设计。
不仅仅是代码洁癖，还是对稳定系统的追求。





### 参考

* [关于质量标准化的思考和实践](https://mp.weixin.qq.com/s/WM9ZOe2Rf1Jn0nmT4MI5Gw)