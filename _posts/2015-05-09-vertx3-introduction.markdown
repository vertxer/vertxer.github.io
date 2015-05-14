---
layout: post
title:  "关于Java框架Vert.x的一点思考"
date:   2015-05-14 14:00:00
categories: intro
---

#Vert.x简介

在Java 20周年之际，Java用户对Java的抱怨与日俱增，比如内存管理、笨重的JavaEE等。而Java依然在[TIOBE编程语言排行榜](http://www.tiobe.com/index.php/content/paperinfo/tpci/index.html)上艰难的维持第一名的位置，随着一些新编程语言的兴起，这个领域目前呈现一种混战的态势。

在这种背景下，Java届的小鲜肉框架-Vert.x，于2015年5月7日发布了3.0-milestone5版本，距离计划6月22日发布Vert.x 3.0.0-final越来越近了，Vert.x用户组的粉丝们近期已经迫不及待的在宇宙中心（注：北京五道口）组织了一次[Vert.x中国用户组Meetup](http://vertxer.org/meetup/2015/04/23/vertx-beijing-first-meetup.html)，针对Vert.x工程化开发问题以及Vert.x3新特性展开了探讨。

Vert.x <http://vertx.io/> 是一个基于JVM的、轻量级的、高性能的应用平台，非常适用于最新的移动端后台、互联网、企业应用架构。Vert.x基于全异步Java服务器Netty，并扩展出了很多有用的特性。Vert.x的亮点有：

* 【同时支持多种编程语言】目前已经支持了Java/JavaScript/Ruby/Python/Groovy/Clojure/Ceylon等。对程序员来说，直接好处是可以使用各种语言丰富的lib，同时也不再为编程语言选型而纠结。
* 【异步无锁编程】经典的多线程编程模型能满足很多web开发场景，但随着移动互联网并发连接数的猛增，多线程并发控制模型性能难以扩展、同时要想控制好并发锁需要较高的技巧，目前Reactor异步编程模型开始跑马圈地，而Vert.x就是这种异步无锁编程的一个首选。
* 【对各种io的丰富支持】目前Vert.x的异步模型已支持TCP, UDP, File System, DNS, Event Bus, Sockjs等
* 【极好的分布式开发支持】Vert.x通过Event Bus事件总线，可以轻松编写分布式解耦的程序，具有很好的扩展性。
* 【生态体系日趋成熟】Vert.x归入Eclipse基金会门下，异步驱动已经支持了Postgres/MySQL/MongoDB/Redis等常用组件。并且有若干Vert.x在生产环境中的应用案例。



#Reactor模式

和传统Java框架的多线程模型相比，Vert.x/Netty是Reactor模式的Java实现。考古了一下Reactor模式，其理论最早由Washington University的Douglas C. Schmidt教授在1995年提出，在这片论文中完整了做了介绍：[Proactor - An Object Behavioral Pattern for Demultiplexing and Dispatching Handlers for Asynchronous Events](http://www.cs.wustl.edu/~schmidt/PDF/proactor.pdf) 。下边对其关键原理部分展开分析：

![Figure 1: Typical Web Server Communication Software Architecture](http://vertxer.org/images/Figure 1- Typical Web Server Communication Software Architect.jpg)
>上图描述了一个经典Web Server在收到Web浏览器请求后的处理过程。

![Figure 2: Multi-threaded Web Server Architecture](http://vertxer.org/images/Figure 2- Multi-threaded Web Server Architecture.jpg)
>上图描述了一个经典Web Server使用多线程模型，并发处理来自多个Web浏览器的请求。

![Figure 3: Client Connects to Reactive Web Server](http://vertxer.org/images/Figure 3- Client Connects to Reactive Web Server.jpg)
>上图描述了Web浏览器连接到一个Reactor模式的Web Server处理过程。利用了Initiation Dispatcher组件，把耗时的io操作事件注册到Initiation Dispatcher组件。

![Figure 4: Client Sends HTTP Request to Reactive Web Server](http://vertxer.org/images/Figure 4- Client Sends HTTP Request to Reactive Web Server.jpg)
>上图描述了Web浏览器访问一个Reactor模式的Web Server处理过程。耗时io操作由其它线程执行，io执行完成后通知Initiation Dispatcher，再回到Http Handler执行。

![Figure 5: Client connects to a Proactor-based Web Server](http://vertxer.org/images/Figure 5- Client connects to a Proactor-based Web Server.jpg)
>上图描述了Web浏览器连接一个Proactor模式的Web Server处理过程。和Reactor的主要区别是耗时io操作交给操作系统异步io库执行（例如GNU/Linux aio），操作系统异步io库执行完毕后，通过异步io通知机制（例如epoll）触发Completion Dispatch，再交给Http Handler执行。

![Figure 6: Client Sends requests to a Proactor-based Web Server](http://vertxer.org/images/Figure 6- Client Sends requests to a Proactor-based Web Server.jpg)
>上图描述了Web浏览器访问一个Proactor模式的Web Server处理过程。和Reactor的主要区别是耗时io操作交给操作系统异步io库执行（例如GNU/Linux aio），操作系统异步io库执行完毕后，通过异步io通知机制（例如epoll）触发Completion Dispatch，再交给Http Handler执行。

事实上，Vert.x/Netty的Reactor实现部分是在Netty 4.0的代码中实现，重要的几个类是io.netty.channel.nio.NioEventLoop, io.netty.channel.epoll.EpollEventLoop, java.nio.channels.spi.SelectorProvider。



#Vert.x 3.0的更新
参考自： <https://groups.google.com/forum/#!msg/vertx/_y_VqFQOVhs/r8zce-zzds0J>

【笔者注】Vert.x3是对Vert.x2的重大升级，不仅仅是package从org.vertx到io.vertx的全面替换，一些重要的核心类也都做了破坏式的重构，几乎很难从vert.x2程序升级到vert.x3程序。下边是Vert.x3的一些功能升级。


Vert.x 2.x中的模块体系去掉了。目前Vert.x 3.0推荐用Maven的模块体系，当然不仅限于Maven。

支持其他语言在Vert.x上的代码生成。

Vert.x 3.0项目构建，从Gradle改为Maven。

为了更好的利用Java8的Lambdas表达式，只支持Java8。

默认采用扁平的classpath结构。

Verticle工厂方式简化。

支持用编程的方式实例化Verticle、以及部署Verticle实例。

当你阻塞Event loop主线程是警告。

移除了platform manager模块。

集群管理可以用编程的方式调用。

支持集群建的共享数据。

完全重写了HTTP client，更完善。

WebSocket API改善。

SSL/TLS的改善。 

Event bus的API改善

支持Event bus代理。

增加了'ext' stack 

增加了MongoService ，支持MongoDB的纯异步驱动。

实现ReactiveStreams 。

This is an implementation of <http://www.reactive-streams.org/>

Options类的使用，可以构建参数进去。

样例工程。<https://github.com/vert-x3/example-proj>


##Vert.xCode location 代码位置
Vert.x 3.0 core工程位于github Eclipse主页: 
<https://github.com/eclipse/vert.x> (master branch) 

Vert.x 3.0的其它工程位于新的Vert-x3 umbrella organisation in 
GitHub: <https://github.com/vert-x3/>

##综述
综合来看，Vert.x3还未正式发版，但其主体功能已经开发完毕并趋于稳定，在应用开发中已经可以考虑使用。程序员在适应了异步回调式的编程方式后，相信很快可能感受到Reactor模式的威力，和Vert.x的魅力。也许Vert.x会给Java和应用框架领域带来惊喜。


##参考资料

1. [High performance reactive applications with Vert.x Tim Fox Red Hat](http://vertxer.org/resource/slide/High%20performance%20reactive%20applications%20with%20Vert.x%20Tim%20Fox%20Red%20Hat.pdf) | by Tim Fox

2. [Vert.x3 Readmap](https://github.com/vert-x3/wiki/wiki/Vert.x-Roadmap)

3. <http://en.wikipedia.org/wiki/Reactor_pattern> 

4. [Reactor - An Object Behavioral Pattern for Concurrent Event Demultiplexing and Dispatching](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.37.9570) | Douglas C. Schmidt, 1995

5. [Proactor - An Object Behavioral Pattern for Demultiplexing and Dispatching Handlers for Asynchronous Events](http://www.cs.wustl.edu/~schmidt/PDF/proactor.pdf) | Irfan Pyarali, Tim Harrison, and Douglas C. Schmidt, 1997

6. [Reactor - an object behavioral pattern for demultiplexing and dispatching handles for synchronous events](http://www.cs.wustl.edu/~schmidt/PDF/reactor-siemens.pdf) | Douglas C. Schmidt, 1997

7. [Architecture of a Highly Scalable NIO-Based Server](https://today.java.net/article/2007/02/08/architecture-highly-scalable-nio-based-server) | Gregor Roth, 2007

8. [Vert.3最新文档预览版](https://vertx.ci.cloudbees.com/view/vert.x-3/job/vert.x3-website/ws/target/site/docs/vertx-core/java/index.html) 
