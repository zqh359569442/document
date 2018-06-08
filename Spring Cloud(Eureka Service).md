# Spring Cloud(Eureka Service)

##  目录
- Eureka 介绍
    - 基本介绍
    - Eureka的优劣势
- Eureka的部署
- Eureka的配置详解
- Eureka的注意点

## 正文

 ![image](image/eureka_architecture.png)


>在上述中引用的我也不知道源头的图片
### Eureka 介绍
&emsp;&emsp;
就上图而言，Eureka其实分为两个部分Eureka Server和Eureka Client.
Eureka  
Eureka Server 需要单独部署，他主要的功能有：服务注册，服务发现，心跳机制，服务缓存，服务剔除。  
- Eureka原理介绍
    - 服务注册  
    Eureka Client 会向Eureka Server 注册自己所提供的服务信息
    - 服务发现  
    Eureka Client 会去Eureka Server 上获取自己的服务列表
    - 心跳机制  
    Eureka Client 每30秒会向Eureka Server 进行心跳检测。这个时间间隔可以修改，但不建议修改
    - 服务缓存  
    Eureka Server 做了一个服务缓存，它会把服务的列表缓存在Eureka Client上，一遍如果Eureka Server服务器瘫痪，那么Eureka Client也会通过缓存服务列表进行RPC调用，但是无法获取到最新服务的变更
    - 服务剔除  
    这个是在心跳机制上制定的  如果心跳检测90秒（也就是3次）检测失败，那么会把这个服务从服务列表中剔除
-  Eureka的优劣势  
&emsp;&emsp;
在spring cloud 的微服务体系中，Eureka的理念不仅仅功能的增强，在我的理解中他更多的体现了一个去中心化的模式(这个在后面的集群化Eureka进行解剖)，拿另外一款服务注册中心ZK进行分析，他是用选举算法进行一个中心化操作。这造成了一部分的资源浪费

### Eureka的部署  
&emsp;&emsp;
在spring官方网站上创建一个简单的 spring boot 项目，地址为 http://start.spring.io/  ，输入相应的信息。同时在Search for Dependencies 中输入 Web， Eureka Server依赖，并创建项目，放入到idea中（其实可以在idea中进行创建，我创建的项目名为：demo-eureka-service,父pom只是用来管理jar包版本的）  
&emsp;&emsp;
先说明下MAVEN依赖
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>  
```
&emsp;&emsp;
主要需要依赖这两个jar包，如果需要可以自行添加  
&emsp;&emsp;
application.properties
```
#服务器部署端口
server.port = 7999
#eureka映射域名，本地所以是localhost
eureka.instance.hostname = localhost
#eureka默认的注册中心
eureka.client.serviceUrl.defaultZone= http://${eureka.instance.hostname}:${server.port}/eureka/

#是否检索服务
eureka.client.fetch-registry= false
#是否向eureka 注册自己，由于自己是eureka server不需要注册自己
eureka.client.register-with-eureka= false
```

&emsp;&emsp;
注意这里有一个坑,应用部署成功后 ，直接http://localhost:7999 ，不需要加/eureka 这个路径。加了访问不成功

&emsp;&emsp;

到了这里，一个简单的eureka server的配置就基本搞定，现在需要告诉spring启动Eureka Server，我们只需要在主类上添加一个注解@EnableEurekaServer 然后启动应用访问地址，当你看到的页面如下，恭喜Eureka Server部署成功
![image](image\eureka_server_html.png)

&emsp;&emsp;  
接下来我们开始尝试去写一个服务提供者，并在eureka server上进行注册。  
&emsp;&emsp;  
创建一个名为：demo-service-provider的项目，并在pom中添加依赖(省去了一些必要的依赖)
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-eureka-server</artifactId>
</dependency>
```
&emsp;&emsp;
并在application.yml中添加
```
server:
  port: 13000
  
spring:
  application:
    name: user-service
  
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:7999/eureka/ 
      #http://eureka server部署的IP：端口号/eureka
```
&emsp;&emsp;  
最后在主类上添加一个@EnableEurekaServer 注解并启动应用，你再次访问你部署的eureka server页面是会看到如下图
![image](image\eureka_server_register.png)
&emsp;&emsp;
那么服务的提供方现在以及注册成功
