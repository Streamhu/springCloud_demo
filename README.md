### 一、基础概念
##### 1. Spring Cloud是一个微服务框架，相比Dubbo等RPC框架, Spring Cloud提供的全套的分布式系统解决方案
##### 2. Spring Cloud对微服务基础框架Netflix的多个开源组件进行了封装，同时又实现了和云端平台以及和Spring Boot开发框架的集成
##### 3. Spring Cloud为微服务架构开发涉及的配置管理，服务治理，熔断机制，智能路由，微代理，控制总线，一次性token，全局一致性锁，leader选举，分布式session，集群状态管理等操作提供了一种简单的开发方式
##### 4. Spring Cloud 为开发者提供了快速构建分布式系统的工具，开发者可以快速的启动服务或构建应用、同时能够快速和云平台资源进行对接
##### 5. Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署
##### 6. Spring Boot 是 Spring 的一套快速配置脚手架，可以基于Spring Boot 快速开发单个微服务，Spring Cloud是一个基于Spring Boot实现的云应用开发工具；Spring Boot专注于快速、方便集成的单个个体，Spring Cloud是关注全局的服务治理框架
##### 7. Spring Cloud的 版本
```
1）Spring Cloud不像其他Spring子项目那样相对独立，它是一个拥有诸多子项目的大型综合项目

2）Spring Cloud可以说是微服务架构解决方案的综合套件组合，其包含的子项目也都独立进行着内容更新与迭代，各自都维护着自己的发布版本号

3）因此每个Spring Cloud版本，包含着多个不同版本的子项目，为了管理每个版本的子项目清单，避免SpringCloud版本号与其子项目版本号混淆，没有采用版本号方式，而是采用命名方式

4）这些版本的名字采用了伦敦地铁站的名字，根据字母表顺序来对应版本时间顺序,如：Angel.SR6,Brixton.SR5,Brixton.SR7,Camden.M1

注意：使用Brixton版本要注意SpringBoot的对应版本，必须要使用1.3.x，而不能使用1.4.x
```
##### 8. 优点：
```
1）集大成者，Spring Cloud 包含了微服务架构的方方面面

2）约定优于配置，基于注解，没有配置文件

3）轻量级组件，Spring Cloud 整合的组件大多比较轻量级，且都是各自领域的佼佼者

4）开发简便，Spring Cloud 对各个组件进行了大量的封装，从而简化了开发

5）开发灵活，Spring Cloud 的组件都是解耦的，开发人员可以灵活按需选择组件
```
##### 9. 缺点：
```
1）项目结构复杂，每一个组件或者每一个服务都需要创建一个项目

2）部署门槛高，项目部署需要配合 Docker 等容器技术进行集群部署，而要想深入了解 Docker，学习成本高	
```

<hr>

### 二、Spring Cloud的子项目

图片【spring cloud子模块】

```
1. Spring Cloud Config	：配置管理工具，支持使用Git存储配置内容，支持应用配置的外部化存储，支持客户端配置信息刷新、加解密配置内容等

2. Spring Cloud Bus	：事件、消息总线，用于在集群（例如，配置变化事件）中传播状态变化，可与Spring Cloud Config联合实现热部署

3. Spring Cloud Netflix ：针对多种Netflix组件提供的开发工具包，其中包括Eureka、Hystrix、Zuul、Archaius等

	1）Spring Cloud Netflix Eureka：一个基于rest服务的服务治理组件，包括服务注册中心、服务注册与服务发现机制的实现，实现了云端负载均衡和中间层服务器的故障转移

	2）Spring Cloud Netflix Hystrix：容错管理工具，实现断路器模式，通过控制服务的节点,从而对延迟和故障提供更强大的容错能力

	3）Spring Cloud Netflix Ribbon：客户端负载均衡的服务调用组件

	4）Spring Cloud Netflix Feign：基于Ribbon和Hystrix的声明式服务调用组件

	5）Spring Cloud Netflix Zuul：微服务网关，提供动态路由，访问过滤等服务

	6）Spring Cloud Netflix Archaius：配置管理API，包含一系列配置管理API，提供动态类型化属性、线程安全配置操作、轮询框架、回调机制等功能

4. Spring Cloud for Cloud Foundry：通过Oauth2协议绑定服务到CloudFoundry，CloudFoundry是VMware推出的开源PaaS云平台

5. Spring Cloud Sleuth：日志收集工具包，封装了Dapper,Zipkin和HTrace操作

6. Spring Cloud Data Flow：大数据操作工具，通过命令行方式操作数据流

7. Spring Cloud Security：安全工具包，为你的应用程序添加安全控制，主要是指OAuth2

8. Spring Cloud Consul：封装了Consul操作，consul是一个服务发现与配置工具，与Docker容器可以无缝集成

9. Spring Cloud Zookeeper：操作Zookeeper的工具包，用于使用zookeeper方式的服务注册和发现

10.Spring Cloud Stream：数据流操作开发包，封装了与Redis,Rabbit、Kafka等发送接收消息

11.Spring Cloud CLI：基于 Spring Boot CLI，可以让你以命令行方式快速建立云组件  
```

<hr>

### 三、spring cloud架构
图片【spring cloud架构图】
```
1. 外部或者内部的非Spring Cloud项目都统一通过API网关（Zuul）来访问内部服务

2. 网关接收到请求后，从注册中心（Eureka）获取可用服务

3. 由Ribbon进行均衡负载后，分发到后端的具体实例

4. 微服务之间通过Feign进行通信处理业务

5. Hystrix负责处理服务超时熔断

6. Turbine监控服务间的调用和熔断相关指标
```

<hr>

### 四、spring cloud代码示例（github地址：https://github.com/Streamhu/springCloud_demo）

项目创建先后顺序：
##### 1. Eureka
```
1) 概念

	（1）Eureka 采用了 C-S 的设计架构

	（2）Eureka Server 作为服务注册功能的服务器，它是服务注册中心。而系统中的其他微服务，使用 Eureka 的客户端连接到 Eureka Server，并维持心跳连接

2）配置	

	（1）eureka.client.register-with-eureka ： 			表示是否将自己注册到Eureka Server，默认为true。

	（2）eureka.client.fetch-registry ：                表示是否从Eureka Server获取注册信息，默认为true。

	（3）eureka.client.serviceUrl.defaultZone ：        设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka ；多个地址可使用 , 分隔
```
##### 2. Provider
##### 3. Consumer
##### 4. Fegin
```
1）概念

	（1）Feign支持服务的调用以及均衡负载
```
##### 5. Hystrix
```
1）概念

	（1）Hystrix的断路器就像我们家庭电路中的保险丝, 一旦后端服务不可用, 断路器会直接切断请求链, 避免发送大量无效请求影响系统吞吐量, 并且断路器有自我检测并恢复的能力

	（2）Fallback相当于是降级操作. 对于查询操作, 我们可以实现一个fallback方法, 当请求后端服务出现异常的时候, 可以使用fallback方法返回的值. fallback方法的返回值一般是设置的默认值或者来自缓存

2）配置

	（1）feign.hystrix.enabled=true	：                   熔断只是作用在服务调用这一端，开启熔断即可 
```
##### 6. Config-server
```
1）概念

	（1）Spring Cloud Config项目是一个解决分布式系统的配置管理方案

	（2）它包含了Client和Server两个部分，server提供配置文件的存储、以接口的形式将配置文件的内容提供出去，client通过接口获取数据、并依据此数据初始化自己的应用

	（3）Spring cloud使用git或svn存放配置文件，默认情况下使用git
```
##### 7. Config-client
##### 8. Zuul
```
1）概念

	（1）在微服务架构中，后端服务往往不直接开放给调用端，而是通过一个API网关根据请求的url，路由到相应的服务

	（2）当添加API网关后，在第三方调用端和服务提供方之间就创建了一面墙，这面墙直接与调用方通信进行权限控制，后将请求均衡分发给后台服务端

2）为什么需要API Gateway

	（1）简化客户端调用复杂度

	（2）数据裁剪以及聚合

	（3）渠道支持

	（4）遗留系统的微服务化改造	
```
##### 9. Zpkin(slueth)
```
1）概念

	（1）Zipkin 是一个开放源代码分布式的跟踪系统，由Twitter公司开源，它致力于收集服务的定时数据，以解决微服务架构中的延迟问题，包括数据的收集、存储、查找和展现

	（2）每个服务向zipkin报告计时数据，zipkin会根据调用关系通过Zipkin UI生成依赖关系图，显示了多少跟踪请求通过每个服务，该系统让开发者可通过一个 Web 前端轻松的收集和分析数据，例如用户每次请求服务的处理时间等，可方便的监测系统中存在的瓶颈

	（3）Zipkin提供了可插拔数据存储方式：In-Memory、MySql、Cassandra以及Elasticsearch。接下来的测试为方便直接采用In-Memory方式进行存储，生产推荐Elasticsearch

```

<hr>

#### **参考网址**
[SpringCloud是什么？](https://www.cnblogs.com/lexiaofei/p/6808152.html)

[SpringCloud学习资料汇总](http://www.ityouknow.com/springcloud/2016/12/30/springcloud-collect.html)

[Spring Cloud 系列文章（纯洁的微笑,最后一篇很多开源项目）](http://www.ityouknow.com/spring-cloud)

[SpringCloud分布式开发五大神兽](https://www.cnblogs.com/ilinuxer/p/6580998.html)


待读：
		2. Spring Cloud 从入门到精通（有一系列）	https://blog.csdn.net/valada/article/details/80892573   （要买的课程，不过其目录可以用来参考）
		7. 使用Spring Cloud与Docker实战微服务       http://book.itmuch.com/
		8. Spring Cloud微服务云应用教程（有系列）   https://www.jdon.com/springcloud.html
		9. Spring Cloud（可以先看这个）             https://springcloud.cc/spring-cloud-dalston.html
		12. Spring Cloud学习路线（最先看这个，方向）https://www.cnblogs.com/moonsoft/p/9360076.html
		13. Spring Cloud基础教程(程序员DD)          https://www.jianshu.com/p/492dfefa2735


<font color=#FF4500 size=3>注：文章是经过参考其他的文章然后自己整理出来的，有可能是小部分参考，也有可能是大部分参考，但绝对不是直接转载，觉得侵权了我会删，我只是把这个用于自己的笔记，顺便整理下知识的同时，能帮到一部分人。
ps : 有错误的还望各位大佬指正,小弟不胜感激</font>