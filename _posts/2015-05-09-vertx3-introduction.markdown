---
layout: post
title:  "Vert.x Introduction for CSDN Java 20 Year Anniversary!"
date:   2015-05-09 14:00:00
categories: intro
---

#Vert.x介绍

在Java 20周年之际，Java用户对Java的抱怨与日俱增，比如内存管理、笨重的JavaEE等。而Java依然在[TIOBE编程语言排行榜](http://www.tiobe.com/index.php/content/paperinfo/tpci/index.html)上艰难的维持第一名的位置，随着一些新编程语言的兴起，这个领域目前呈现一种混战的态势。

在这种背景下，Java届的小鲜肉框架-Vert.x，于2015年5月7日发布了3.0-milestone5版本，距离计划6月22日发布Vert.x 3.0.0-final越来越近了，Vert.x用户组的粉丝们近期已经迫不及待的在宇宙中心（注：北京五道口）组织了一次[Vert.x中国用户组Meetup](http://vertxer.org/meetup/2015/04/23/vertx-beijing-first-meetup.html)，针对Vert.x工程化开发问题以及Vert.x3新特性展开了探讨。

Vert.x <http://vertx.io/> 是一个基于JVM的、轻量级的、高性能的应用平台，非常适用于最新的移动端后台、互联网、企业应用架构。Vert.x基于全异步Java服务器Netty，并扩展出了很多有用的特性。Vert.x的亮点有：

* 【同时支持多种编程语言】目前已经支持了Java/JavaScript/Ruby/Python/Groovy/Clojure/Ceylon等。对程序员来说，直接好处是可以使用各种语言丰富的lib，同时也不再为编程语言选型而纠结。
* 【异步无锁编程】经典的多线程编程模型能满足很多web开发场景，但随着移动互联网并发连接数的猛增，多线程并发控制模型性能难以扩展、同时要想控制好并发锁需要较高的技巧，目前Reactor异步编程模型开始跑马圈地，而Vert.x就是这种异步无锁编程的一个首选。
* 【对各种io的丰富支持】目前Vert.x的异步模型已支持TCP, UDP, File System, DNS, Event Bus, Sockjs等
* 【极好的分布式开发支持】Vert.x通过Event Bus事件总线，可以轻松编写分布式解耦的程序，具有很好的扩展性。
* 【生态体系日趋成熟】Vert.x归入Eclipse基金会门下，异步驱动已经支持了Postgres/MySQL/MongoDB/Redis等常用组件。并且有若干Vert.x在生产环境中的应用案例。




#Reactor模式

考古了一下Reactor模式，其理论最早由Washington University的Douglas C. Schmidt教授在1995年提出，在这片论文中完整了做了介绍：[Proactor - An Object Behavioral Pattern for Demultiplexing and Dispatching Handlers for Asynchronous Events](http://www.cs.wustl.edu/~schmidt/PDF/proactor.pdf) 。下边对其展开分析：

![Figure 1: Typical Web Server Communication Software Ar- chitecture](http://vertxer.org/images/Figure 1- Typical Web Server Communication Software Ar- chitecture.jpg)
>上图是

![Figure 2: Multi-threaded Web Server Architecture](http://vertxer.org/images/Figure 2- Multi-threaded Web Server Architecture.jpg)
>上图是

![Figure 3: Client Connects to Reactive Web Server](http://vertxer.org/images/Figure 3- Client Connects to Reactive Web Server.jpg)
>上图是

![Figure 4: Client Sends HTTP Request to Reactive Web Server](http://vertxer.org/images/Figure 4- Client Sends HTTP Request to Reactive Web Server.jpg)
>上图是

![Figure 5: Client connects to a Proactor-based Web Server](http://vertxer.org/images/Figure 5- Client connects to a Proactor-based Web Server.jpg)
>上图是

![Figure 6: Client Sends requests to a Proactor-based Web Server](http://vertxer.org/images/Figure 6- Client Sends requests to a Proactor-based Web Server.jpg)
>上图是




#Vert.x 3.0的更新
参考自： <https://groups.google.com/forum/#!msg/vertx/_y_VqFQOVhs/r8zce-zzds0J>

【笔者注】Vert.x3是对Vert.x2的重大升级，不仅仅是package从org.vertx到io.vertx的全面替换，一些重要的核心类也都做了破坏式的重构，几乎很难从vert.x2程序升级到vert.x3程序。下边是Vert.x3的一些功能升级。



###No more module system
***The Vert.x module system has gone***. Now you create your verticles and 
package them into standard java jars. These jars can be pushed to Maven 
repositories (or bintray) just like any Maven artifact. 

When you use these artifacts in your application you specify them as 
standard Maven dependencies in your project and they are resolved at 
build-time, not at run-time like in Vert.x 2.0. 

Non Java verticles (e.g. JavaScript) can also be packaged in jars and 
pushed as Maven artifacts. We will also support resolving in other ways 
and from other places (e.g. at run-time and from npm modules) before 
3.0.final. 

We recommend package your application into [a set of] of fat executable 
jars which contain all the dependencies they need to run. The example 
project (see bottom of this document) shows an example of that. 

###Code generation of other language APIs 

One of the big pain points of Vert.x 2.x development was keeping all the 
other language APIs in sync as changes occur in the Java core APIs, as 
it was a manually process. As we have a small team, this was becoming 
unsustainable and hard to manage. 

In Vert.x 3.0 we maintain just the core Java API and we will generate 
other language APIs from that. 

Generation works by inspecting the Java source of the core APIs before 
compile and constructing an internal model from that. This is then fed 
into a MVEL2 template (one for each language) which then outputs the 
code in other languages. 

In order for codegen to be feasible we specify a set of constraints that 
any API that we run codegen on must follow. The codegen project details 
these constraints: 

<https://github.com/vert-x3/codegen>

So far we have JavaScript and Groovy more or less complete, and Julien 
is currently working on Ceylon. Other languages will follow. 

###Use of Maven for the Vert.x builds 

We now use Maven, not Gradle to build the Vert.x main project and the 
"ext" projects. Please note that we do not mandate Maven for your own 
Vert.x 3.0 projects - you shoud be able to use whatever build tool you 
want for that (e.g. Gradle). Vert.x 3.0 projects generally have a very 
simple setup (much simpler than Vert.x 2.0). 

###Java 8 only 

Vert.x 3.0 is Java 8 only. This is to take advantage of new language 
features in Java 8, the most important of which is Lambdas which make 
developing against event based APIs so much nicer than in previous 
versions of Java. We also chose Java8 so we can use Nashorn - the new 
high performance JavaScript engine that it contains. 

###Flat classpath by default 

When deploying verticles, there is a simple single flat classpath (by 
default). This should greatly simplify embedding and debugging. If you 
do wish to deploy verticles with classloader isolation this is still 
available as an option, but it's considered an edge case now. 

###Verticle factory simplification 

There is no more `langs.properties`. Verticle factories are loaded from 
the classpath using the service loader pattern. Verticle factories can 
also be registered and unregistered programmatically. 

###Deployment of Verticle instances 

You can now instantiate verticles and deploy verticle *instances* 
programmatically. 

###Warnings if you block an event loop 

You will now see warnings in the logs if you inadvertently (or 
deliberately!) block an an event loop, so you can identify and fix the 
issue more quickly. You will also see warnings if you block a worker 
thread for too long. 

###Removal of platform manager 

The platform manager API has been removed, and methods for deploying 
verticles have moved to the Vertx interface. The API for deploying 
verticles is much simpler, so this should simplify things when embedding. 

###Programmatic cluster manager 

Cluster managers can now be specified programmatically 

###Clustered shared data 

Vert.x now supports a distributed Map API which works across the 
cluster. You can put data in at one Vert.x node and read it from another. 

We also provide an asynchronous cluster wide lock implementation, and 
asynchronous cluster-wide counters. These are very useful building 
blocks for distributed applications. 

###Completely rewritten HTTP client 

The HTTP client has been completely rewritten with much saner behaviour 
for opening and closing connections, pipe-lining and keep alive. 

###WebSocket API improvements 

The WebSocket API has been improved so you can now send and receive 
WebSocket frames. 

###Improvements to SSL/TLS 

The new SSL API allows you to configure certificates and keys in several 
ways, not just with Java key stores. For example, with Strings and 
Buffers (similar to Node.js). 

We also support certificate revocation lists and specification of cipher 
suites. 

###Event bus API improvements 

When registering a handler you now receive a Registration object which 
you can later use to unregister. 

We also now support custom codecs on the event bus so you can send any 
object you want across the event bus (e.g. a POJO) as long as you write 
a codec for it. We also support receiving messages as a different type 
to what was sent, e.g. you could send a buffer which contains some BSON 
and receive it as a POJO representing that BSON. 

###Event bus proxies 

You can register service interfaces with the event bus, then you can 
interact with the service from a completely different verticle using a 
proxy for the service. 

###The 'ext' stack 

https://github.com/vert-x3/ext 

Vert.x extensions live in the `ext` project. This is functionality which 
is not part of Vert.x core but contains useful stuff that people need to 
write real applications with Vert.x. You can think of this plus Vert.x 
core comprising the Vert.x "stack". 

So far, ext just contains a few things including: SockJS, RouteMatcher, 
and MongoService and Reactivestreams but will contain many other 
services before 3.0.final. 

A lot of the work in Vert.x 3.0 is in creating this stack. 

###MongoService 

This is a component of the ext stack. It's an API that lets you interact 
with a MongoDB database. It uses the experimental 100% Async driver from 
the MongoDB folks. You can deploy it as a verticle and interact with it 
over the event bus using a proxy. 

###ReactiveStreams 

This is an implementation of <http://www.reactive-streams.org/>

###Use of options classes 

We now use options classes widely throughout Vert.x in order to 
configure objects. This means a lot of the setter methods on interfaces 
such as NetClient, NetServer, HttpClient and HttpServer have been removed. 

###Example project 

I've put together an example Maven project which shows how you might 
write a simple application with Vert.x 3.0. 

It's a simple web application that uses SockJS and also has a REST API. 
It has a JavaScript main verticle which starts up the app and a Java 
WebServer and a Groovy stock ticker. 

<https://github.com/vert-x3/example-proj>

##What's next? 

** Most of Vert.x 3.0 core is done but there is a some more work to do 
** More work on codegen 
** Codegen templates for other language APIs (Scala, Ruby etc) 
** Documentation, including JavaDoc/Asciidoc. 
** The rest of the ext stack. This is where most of the work is. 


##Vert.xCode location 代码位置
Vert.x 3.0 core工程位于github Eclipse主页: 
<https://github.com/eclipse/vert.x> (master branch) 

Vert.x 3.0的其它工程位于新的Vert-x3 umbrella organisation in 
GitHub: <https://github.com/vert-x3/>


##参考资料

1. [High performance reactive applications with Vert.x Tim Fox Red Hat](http://vertxer.org/resource/slide/High%20performance%20reactive%20applications%20with%20Vert.x%20Tim%20Fox%20Red%20Hat.pdf) | by Tim Fox

2. [Vert.x3 Readmap](https://github.com/vert-x3/wiki/wiki/Vert.x-Roadmap)

3. <http://en.wikipedia.org/wiki/Reactor_pattern> 

4. [Reactor - An Object Behavioral Pattern for Concurrent Event Demultiplexing and Dispatching](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.37.9570) | Douglas C. Schmidt, 1995

5. [Proactor - An Object Behavioral Pattern for Demultiplexing and Dispatching Handlers for Asynchronous Events](http://www.cs.wustl.edu/~schmidt/PDF/proactor.pdf) | Irfan Pyarali, Tim Harrison, and Douglas C. Schmidt, 1997

6. [Reactor - an object behavioral pattern for demultiplexing and dispatching handles for synchronous events](http://www.cs.wustl.edu/~schmidt/PDF/reactor-siemens.pdf) | Douglas C. Schmidt, 1997

7. [Architecture of a Highly Scalable NIO-Based Server](https://today.java.net/article/2007/02/08/architecture-highly-scalable-nio-based-server) | Gregor Roth, 2007
