---
layout: post
title:  "Vert.x Introduction!"
date:   2015-05-09 14:00:00
categories: meetup
---

#Vert.x Introduction
##Reactor模式
Event loop + Work pool
###Reactor / Proactor

<http://en.wikipedia.org/wiki/Reactor_pattern>

[Reactor - An Object Behavioral Pattern for Concurrent Event Demultiplexing and Dispatching](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.37.9570) | Douglas C. Schmidt, 1995

[Proactor - An Object Behavioral Pattern for Demultiplexing and Dispatching Handlers for Asynchronous Events](http://www.cs.wustl.edu/~schmidt/PDF/proactor.pdf) | Irfan Pyarali, Tim Harrison, and Douglas C. Schmidt, 1997

[Reactor - an object behavioral pattern for demultiplexing and dispatching handles for synchronous events](http://www.cs.wustl.edu/~schmidt/PDF/reactor-siemens.pdf) | Douglas C. Schmidt, 1997

[Architecture of a Highly Scalable NIO-Based Server](https://today.java.net/article/2007/02/08/architecture-highly-scalable-nio-based-server) | Gregor Roth, 2007




##Netty
<https://github.com/netty/netty>

##Vert.x 2.x源码分析
<https://github.com/eclipse/vert.x>

##工程化最佳实践探讨
//TODO

#Vert.x vs Undertow
1. From techempower's round 1 / round 8 / round 9 / round 10 benchmark result, undertow is very good (Both Throughout and Latency are excellent), and vert.x performance test results immediately follow undertow. But round9 JSON serialization tests show that undertow have certain error rate, so probably concluded that: vert.x performance slightly below undertow, but the stability is better than undertow.
<https://www.techempower.com/benchmarks/previews/round10/>
<https://www.techempower.com/benchmarks/>
>Both Undertow and Vert.x's performance is way better than most people actually need. However we wouldn't recommend choosing a product on the basis of a few %age points of difference, usually there are much more important factors...

2. From an architectural point of view vert.x and undertow are both non-block architecture, but vert.x is more suitable for Distributed System (event bus), and undertow is suitable for the embedded application (XNIO worker instance). Our security gateway product scene is distributed, it is more appropriate vert.x.
<Http://vertx.io/manual.html>
<Http://undertow.io/documentation/core/overview.html>
>Vert.x does a *lot* more than Undertow. Undertow was originally designed as the replacement for the webserver in JBoss app server and that's its main purpose. Vert.x has a lot more functionality - UDP, DNS, TCP, File System, Event Bus, Sockjs, etc etc, etc. Also vert.x is polyglot (not just Java)

3. Community activity and components maturity, vert.x win (4 times ahead). At the same time the number of staff involved from the development perspective, in April of this year vert.x dev mail list has 215 posts, while only 16 posts undertow, vert.x win 15 times ahead.
>Yes, Vert.x is much more popular than Undertow (compare the number of stars we have on GitHub too :) ), and we have a much more active development community.

4. from the health of the Road Map view, undertow belong JBoss Community (with my experience in the years since JBoss is an open source project launched most bland, such as JBoss Seam that been designed for chanlleging Spring Framework, lost finally), and has vert.x defected to the Eclipse Foundation (more powerful organization, behind IBM, at the same time Vert.x's former owner - VMware also supports Vert.x).
>Red Hat employees most of the core engineers on _both_ Vert.x and Undertow. But Vert.x is the *official* reactive framework for Red Hat, and Vert.x is also an Eclipse project yes.

#Vert.x 3.0 update
<https://groups.google.com/forum/#!msg/vertx/_y_VqFQOVhs/r8zce-zzds0J>

###Code location 
Vert.x 3.0 core is in the main Eclipse project: 
<https://github.com/eclipse/vert.x> (master branch) 

The rest of Vert.x 3.0 is under the new Vert-x3 umbrella organisation in 
GitHub: <https://github.com/vert-x3/>

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

##Vert.x Roadmap
<https://github.com/vert-x3/wiki/wiki/Vert.x-Roadmap>

###Vert.x 3.0.0-milestone4 (release: 07 April 2015)

###Vert.x 3.0.0-milestone5 (release: 07 May 2015)
Services:

 * Email - 1
 
Other:

* dynamic service factories from bintray (like maven-service-proxy) - 1
* maven service factory should support zip artifacts - 1

Languages:

* JRuby implementation (including codetrans) - 1
* Tweak Groovy API - 1
* Require NPM modules from Node path - 1

Documentation

* cluster configuration doc - 1
* finish off core docs - 1
* vert.x 2 migration doc - 2

Examples:

* end to end examples - port old vtoons example - 1
* more database examples - raw jdbc? - 1
* make sure examples translate ok - 1
* tutorials

###Vert.x 3.0.0-milestone6 (release: 28 May 2015) Feature complete
Services:

* Work queue - 2
* Redis - 3
* AMQP 1.0 - 3
* RabbitMQ - 3

Other:

* JGroups cluster manager - 2
* OpenShift example / support - 1
* Docker support - 2
* Fabric8 integration/example

###Vert.x 3.0.0-final (release: 22 June 2015)
* Fix bugs
* Tweaks
* Feedback from community
* Performance tuning


##Vert.x 3.x源码分析
//TODO