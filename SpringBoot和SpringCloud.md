## SpringBoot和SpringCloud

#### 1.什么是SpringBoot
Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。

Spring Boot简化了Spring众多框架中所需的大量且繁琐的配置文件，所以 SpringBoot是一个服务于框架的框架，服务范围是简化配置文件。

SpringBoot最明显的特点是，让文件配置变的相当简单、让应用部署变的简单（SpringBoot内置服务器，并装备启动类代码），可以快速开启一个Web容器进行开发。

##### Spring Boot的核心功能

1、 可独立运行的Spring项目：Spring Boot可以以jar包的形式独立运行。

2、 内嵌的Servlet容器：Spring Boot可以选择内嵌Tomcat、Jetty或者Undertow，无须以war包形式部署项目。

3、 简化的Maven配置：Spring提供推荐的基础POM文件来简化Maven 配置。

4、 自动配置Spring：Spring Boot会根据项目依赖来自动配置Spring 框架，极大地减少项目要使用的配置。

5、 提供生产就绪型功能：提供可以直接在生产环境中使用的功能，如性能指标、应用信息和应用健康检查。

6、 无代码生成和xml配置：Spring Boot不生成代码。完全不需要任何xml配置即可实现Spring的所有配置。

#### 2.SpringBoot自动配置原理
Spring Boot有一个全局配置文件：application.properties或application.yml

Spring Boot的启动类上有一个@SpringBootApplication注解，这个注解是Spring Boot项目必不可少的注解

@SpringBootApplication是一个复合注解或派生注解，在@SpringBootApplication中有一个注解@EnableAutoConfiguration，开启自动配置。而这个注解也是一个派生注解，其中的关键功能由@Import提供，其导入的AutoConfigurationImportSelector的selectImports()方法通过SpringFactoriesLoader.loadFactoryNames()扫描所有具有META-INF/spring.factories的jar包。spring-boot-autoconfigure-x.x.x.x.jar里就有一个这样的spring.factories文件。

这个spring.factories文件也是一组一组的key=value的形式，其中一个key是EnableAutoConfiguration类的全类名，而它的value是一个xxxxAutoConfiguration的类名的列表，这些类名以逗号分隔，如下图所示：

![avatar](https://img-blog.csdnimg.cn/20181107130442565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ3NDUwNjk=,size_16,color_FFFFFF,t_70)

这个@EnableAutoConfiguration注解通过@SpringBootApplication被间接的标记在了Spring Boot的启动类上。在SpringApplication.run(...)的内部就会执行selectImports()方法，找到所有JavaConfig自动配置类的全限定名对应的class，然后将所有自动配置类加载到Spring容器中。

#### 3.SpringBoot读取配置文件的方式

直接使用@Value注解@Value（“${}”）,默认读取application.yml中的配置信息。

@ConfigurationProperties注解提供了我们将多个配置选项注入复杂对象的能力。它要求我们指定配置的共同前缀。这种形式支持从驼峰camel-case到短横分隔命名kebab-case的自动转换。默认读取application.yml中的配置信息

    单独使用@ConfigurationProperties的话依然无法直接使用配置对象，因为它并没有被注册为Spring Bean。我们可以通过两种方式来使得它生效。
    
    可以使用@Component、@Configuration等注解注入Spring IoC使之生效。
    
    还可以直接使用注解@EnableConfigurationProperties进行注册，这样就不需要显式声明配置类为Spring Bean了。
    
使用 @PropertySource 注解加载指定的配置文件

总结：
使用@Value注解可以直接读取application.yml配置文件中的配置信息。

使用@ConfigurationProperties也可以读取application.yml文件中的配置信息。

使用@ConfigurationProperties和@PropertySource可以读取指定properties配置文件中的配置信息。

#### 4.什么是微服务架构
SOA （Service Oriented Architecture）是一种面向服务的架构，基于分布式架构，它将不同业务功能按照更细粒度的服务进行拆分，并通过服务之间的接口和协议进行通信。微服务架构基于SOA架构，是SOA的一种轻量级解决方案，是SOA的升华。微服务架构强调的重点在于，业务系统需要彻底地组件化和服务化，原有的单个业务系统会拆分为多个可以独立开发、设计、运行和运维的小应用（微服务）。这些服务基于业务能力构建，并通过自动化机制进行独立部署，同时这些微服务组件使用不同的编程语言实现，使用不同的数据存储技术，并保持最低限度的集中式管理。

#### 5.Ribbon和Feign的区别
Ribbon：Ribbon 是一个基于 HTTP 和 TCP 客户端的负载均衡器它可以在客户端配置 ribbonServerList（服务端列表），然后轮询请求以实现均衡负载。它在联合 Eureka 使用时ribbonServerList 会被 DiscoveryEnabledNIWSServerList 重写，扩展成从 Eureka 注册中心获取服务端列表同时它也会用 NIWSDiscoveryPing 来取代 IPing，它将职责委托给 Eureka 来确定服务端是否已经启动。 

Feign 是在 Ribbon的基础上进行了一次改进，是一个使用起来更加方便的 HTTP 客户端。采用接口的方式， 只需要创建一个接口，面向接口；然后在上面添加注解即可 ，将需要调用的其他服务的方法定义成抽象方法即可， 不需要自己构建http请求。然后就像是调用自身工程的方法调用，而感觉不到是调用远程方法，使得编写 客户端变得非常容易。

#### 6.SpringCloud断路器的作用
当一个服务调用另一个服务由于网络原因或自身原因出现问题，调用者就会等待被调用者的响应

当更多的服务请求到这些资源导致更多的请求等待，发生连锁效应（雪崩效应）
断路器有完全打开状态:一段时间内 达到一定的次数无法调用 并且多次监测没有恢复的迹象 断路器完全打开 那么下次请求就不会请求到该服务

半开:短时间内 有恢复迹象 断路器会将部分请求发给该服务，正常调用时 断路器关闭

关闭：当服务一直处于正常状态 能正常调用

#### 7.为什么要用SpringBoot
##### 1. SpringBoot使编码变得简单
建一个web项目，通常需要添加多个依赖，而SpringBoot则会帮助开发着快速启动一个web容器，我们只需要在pom文件中添加如下一个starter-web依赖即可。
##### 2. SpringBoot使配置变得简单
Spring Boot更多的是采用Java Config的方式，对Spring进行配置。
##### 3.SpringBoot使部署变得简单
一键启动，解压jar，运行jar；

不需要预部署到应用服务器；

降低对运行环境的基本要求，环境变量中有JDK即可。
##### 4. SpringBoot使监控变得简单
采用了spring-boot-start-actuator之后，直接以REST的方式，获取进程的运行期性能参数。
#### 8.SpringBoot的核心配置文件有哪几个，区别是什么
SpringBoot的核心配置文件有application和bootstarp配置文件。

application：文件主要用于Springboot自动化配置文件。

bootstarp:
使用Spring Cloud Config注册中心时 需要在bootStarp配置文件中添加链接到配置中心的配置属性来加载外部配置中心的配置信息。

#### 9.SpringBoot的配置文件有哪几种格式，区别是什么
SpringBoot中的配置文件主要有三种格式，properties、yaml、和xml方式

使用最广泛的配置文件是yaml，yaml之所以流行，除了他配置语法精简之外，还因为yaml是一个跨编程语言的配置文件，.yml采取的是缩进的格式 不支持@PropertySource注解导入配置

xml目前很少用到

#### 10.SpringBoot的核心注解是哪个，主要由哪几个注解组成
启动类上注解@SpringBootApplication，是 Spring Boot 的核心注解，主要包含了以下 3 个注解：

@SpringBootConfiguration：组合了 @Configuration 注解，实现配置文件的功能。

@EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置功能：             @SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })。

@ComponentScan：Spring组件扫描。

#### 11.开启SpringBoot的特性有哪几种方式

    继承spring-boot-starter-parent项目
    导入spring-boot-dependencies项目依赖
    
#### 12.SpringBoot需要独立的容器运行吗
可以不需要，内置了 Tomcat/ Jetty 等容器。

#### 13.运行SpringBoot有哪几种方式
打包用命令或者放到容器中运行

用 Maven/Gradle 插件运行

直接执行 main 方法运行

#### 14.如何理解SpringBoot中的starters
Starters可以理解为启动器，它包含了一系列可以集成到应用里面的依赖包，你可以一站式集成Spring及其他技术，而不需要到处找示例代码和依赖包。如你想使用Spring JPA访问数据库，只要加入springboot-starter-data-jpa启动器依赖就能使用了。Starters包含了许多项目中需要用到的依赖，它们能快速持续的运行，都是一系列得到支持的管理传递性依赖。

#### 15.如何在SpringBoot启动时运行特定的代码
实现ApplicationRunner接口

实现CommandLineRunner接口

可以设置Order来设定执行的顺序

#### 16.SpringBoot实现热部署有哪几种方式
##### spring-boot-devtools

引用devtools依赖，org.springframework.boot spring-boot-devtools,在配置文件application.yml中添加配置。当修改一个java类时就会热更新。

##### 开发工具idea中使用JRebel插件

#### 16.SpringBoot多套不同环境如何配置
多套不同环境(development、test、production)配置，我们基于SpringBoot的Profiles来实现。**profile配置方式有两种：**
- 多profile文件方式：提供多个配置文件，每个代表一种环境。
- application-dev.properties/yml 开发环境
- application-test.properties/yml 测试环境
- application-pro.properties/yml 生产环境
- yml多文档方式：在yml中使用 --- 分隔不同配置 

**profile激活三种方式：**
- 配置文件： 在配置文件中配置：spring.profiles.active=dev
- 虚拟机参数：在VM options 指定：-Dspring.profiles.active=dev
- 命令行参数：java –jar xxx.jar --spring.profiles.active=dev
 
一般我们将生产环境的配置文件放到生产环境的服务器中，以固定命令执行启动：java -jar myboot.jar --spring.config.location=/xx/yy/xx/application-prod.properties。或者使用Jenkins在执行打包，配置上maven profile功能，使用服务器的配置文件。或者使用配置中心管理配置文件。

#### 17.SpringBoot可以兼容老spring项目吗，如何做
可以兼容，使用 @ImportResource 注解导入老 Spring 项目配置文件。

#### 18.什么是SpringCloud

- SpringCloud 是一个微服务框架, 相比Dubbo等RPC框架, SpringCloud提供了全套分布式系统解决方案。

- SpringCloud对微服务基础框架Netflix的多个开源组件进行了封装, 同时又实现了云端平台以及和SpringBoot开发框架的集成。
- SpringCloud为开发者提供了快速构建分布式系统的工具, 开发者可以快速的启动服务或构建应用, 同时能够快速和云平台资源进行对接。
- SpringCloud为微服务架构开发涉及的配置管理, 服务治理, 熔断机制, 智能路由, 微代理, 控制总线, 一次性token, 全局一致性锁, leader选举, 分布式session, 集群状态管理等操作提供了一种简单地开发方式

#### 19.SpringCloud常用组件
##### 技术组件：
- 注册中心：Eureka，服务之间互相发现
- 客户端负载均衡：Ribbon
- 声明式远程方法调用：OpenFeign
- 服务降级、熔断：Hystrix
- 网关：Zuul

#### 20.SpringCloud如何实现服务注册的
1.服务发布时，指定对应的服务名,将服务注册到 注册中心(eureka zookeeper)

2.注册中心加@EnableEurekaServer,服务用@EnableDiscoveryClient，然后用ribbon或feign进行服务直接的调用发现。

#### 21.SpringCloud中Ribbon的主要作用
Ribbon是一组基于NetflixRibbon的客户端负载平衡工具,主要功能是提供客户端软件负载平衡算法，并将Netflix的中间层服务连接在一起。负载均衡将用户的请求平均分配给多个服务，以实现系统的HA。