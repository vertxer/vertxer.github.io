# Vertx-rpc框架 #

by 刘小溪

### Vert.x的微服务
最近关于微服务的概念到处都在宣传,而Vert.x的verticle本身就是很好的一种服务定义,你可以把verticle看成一个service,小一点,你可以把verticle看成一个actor.
这样你的视角会切到Actor模型里.这里我们讨论如何用vert.x实现基于接口的服务化.

传统Java开发人员受EJB以及Spring的影响比较深,所以对面向接口编程了解的比较多.哪怕跨JVM也可以通过接口来调用对方提供的方法.这是非常友好方便的开发方式,因为框架层面做了服务发现
以及服务生命周期的管理.

Vert.x本质上可以通过verticle来定义服务边界,通过EventBus来包装并提供服务,消费端可以根据EventBus的Address来调用暴露出来的服务.而服务的发现以及隐藏则由Vert.x包装掉,乍一看
一切都显得很自然,仿佛Vert.x就是为微服务而生的.

但是Vert.x的EventBus有两点不太好的地方,导致不能原生支持面向接口的服务调用.


### EventBus的两宗罪
EventBus是Vert.x的核心,因为它的存在可以使得系统模块化解耦成为可能,同时也可以将业务水平扩展.我们只需要定义EventBus地址然后传入Json对象就可以,所有的事情都很流畅.
但是这里有一点不太好的地方,默认传输协议走Json.而Json本身不太好定义数据类型.

如果了解Vert.x历史的同学肯定知道为什么EventBus一定要使用Json作为主要的传输对象——为了兼容其他JVM上的弱语言.
可是现实情况中80%的项目都是基于Java跑的,根本就没有考虑去兼容其他JVM上的语言.这就造成了一种尴尬,大家都传Json对象,然后在Handler里先做一次转换,将Json对象转换成POJO,之后再做业务上的逻辑.
这里明显在`Handler`里做了很多与业务无关的事情,服务定位与对象转换,这些本质上应该是框架层面去解决的.

不能隐试的将Json对象转成POJO这是EventBus的`一宗罪`.(`Vert.x3可以直接接受POJO,但是针对一个address,只能接收一种POJO格式.`)
另外当我们使用EventBus的时候不能很方便的确定方法逻辑,简单的讲我只能`EventBus.send()`,后面跟地址参数,以及JSON对象,而这并不适合描述业务,这是EventBus的`二宗罪`.

举个例子,有一个业务逻辑,对用户的账户进行存款与取款这两个操作,如果用Java接口来描述就很Easy.

```java
interface Account {
void saveMemory(int total);
void withdrawMemory(int total);
}
```

如果换成`EventBus`你会发现没办法定义行为的名称(`java的方法`),只能通过`eb.send("address", json)`来调用目标接口.
这里的json也许得包含`method`的描述.这样会显得很啰嗦,关键是对IDE不友好,重构起来不方便,严重影响团队开发流畅度.

看到这里大家也许已经有点明白了,如果能够做到RPC调用,那会非常方便我们开发.

### Vertx-RPC介绍
介于上面的原因,我们开发了一个简单的RPC框架,它简单的封装了EventBus,使之可以包装成Java的接口.这样客户端与服务端之间调用就像本地接口一样.
[vertx-rpc](https://github.com/LeapAppServices/vertx-rpc)其实做的非常简单,他只依赖protostuff,作为数据传输的协议,当然也可以使用JSON协议.
接着只需要定义好接口,然后在服务端实现接口,而客户端只依赖接口,单独将项目打包成jar暴露给出来就可以使用了.

我们继续上面的例子,根据接口我们会把项目分成两个Maven模块.`SPI`,`SPI-impl`,在parent的`pom.xml`定义好即可.
下面我们先在`impl`的项目里启动好`service`并通过`EventBus`暴露服务.

```java

Vertx vertx = Vertx.vertx();
String address = "serviceAddress";

RPCServerOptions rpcServerOptions = new RPCServerOptions(vertx).setBusAddress(address).addService(new MyServiceImpl());
new VertxRPCServer(rpcServerOptions);
```

以上四行代码就已经成功的定义了服务,是这里要注意`new MyServiceimpl()`必须实现`spi`项目里的`MyService`接口.下面我们来看调用方,怎么调用服务.

```java
Vertx vertx = Vertx.vertx();
String address = "serviceAddress";

RPCClientOptions<MyService> rpcClientOptions = new RPCClientOptions<MyService>(vertx).setBusAddress(address).setServiceClass(MyService.class);
MyService myService = new VertxRPCClient<>(rpcClientOptions).bindService();
```

这里本质上是两行代码,一行定义了rpcClient的配置通过`RPCClientOptions`,另外一行直接绑定服务,通过`VertxRPCClient.bindService()`.
最后得到了接口`MyService`.下面你就可以在客户端直接调用服务端的接口了,一切又回到了以前的面向接口编程 :)


这里有个完整的[项目例子](https://github.com/stream1984/vertx-rpc-example).我们把项目里的`SPI`定义接口,里面除了接口,还包含接口用到的对象以及异常.
以后所有的操作都是面向这个SPI来做,`service-impl`就实现了其接口,`outer-invoker`其实是外部的service依赖接口来调用`service-impl`.这便是完全面向接口编程.

vertx-rpc除了包装EventBus使之可以通过Java标准方法调用外,还可以通过注解来定义每一个方法的超时时间.

```java
@RequestProp(timeout = 1, timeUnit = TimeUnit.SECONDS, retry = 2)
void hello(String what, Handler<AsyncResult<String>> handler);
```

这里就对hello方法定义了超时时间,以及超时重试次数,非常方便用于幂等的方法.
上面的方法第二个参数是Vert.x的handler回调,这里也可以使用`RxJava`或者`Java8的CompletableFuture`

```java
Observable<String> hello(String what)

//或者

CompletableFuture<String> hello(String what)

```
具体可以参考[项目例子](https://github.com/stream1984/vertx-rpc-example)

### 作者简介 ###

![](./photo_liuxiaoxi.jpg)

刘小溪

leap.as高级Java工程师, 关注Java异步高性能框架Vert.x以及容器化技术Docker,喜欢函数式语言.


----------

PS：对Vert.x感兴趣的开发者朋友，可以移步去由炼石网络公司主办的[Vert.x Meetup#3](http://www.meetup.com/Vertx-Beijing/events/224580619/)，现场与大牛们零距离探讨技术问题，聆听Vert.x的最佳实践，您也可以成为vert.x模块的贡献者。

议程 | Agenda：

14:00 - 14:15 暖场 | Intro

14:15 - 15:30 Keynote: 三年Vert.x实践之路分享 | three years of Vert.x practice | by 刘小溪 (PaaS startup leap.as 架构师)

15:30 - 15:40 茶歇 | Break

15:40 - 16:10 Keynote: Vert.x性能优化 | Vert.x performance optimization | by 刘晓辉(运动家app架构师)

16:10 - 16:40 闪电演讲 | Lightning talks

16:40 - 17:00 自由交流 | Free talk

费用：AA制，人均30元RMB（用于场地、饮料及干果） | Fee: AA system, the per capita 30 yuan RMB (for space and fruit, dried fruit, water).