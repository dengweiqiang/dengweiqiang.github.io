---
title: Spring Cloud 基础
copyright: true
date: 2020-05-06 18:11:01
tags: [学习]
---

一个分布式系统应该要有这些基础组件，注册中心，负载均衡，熔断器，网关，配置中心，SpringCloud在SpringBoot的基础上，实现了一套开箱即用的分布式系统架构，我对SpringCloud的这些基础应用的使用，做了一个汇总，主要是能快速让一个分布式应用跑起来。

<!--more-->

### Eureka

#### 服务端

1. 依赖

   ```xml
   <!--eureka server -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka-server</artifactId>
   </dependency>
   ```

2. 配置文件

   ```properties
   # 应用名称
   spring.application.name=eureka-server
   # 服务端口
   server.port=8761
   # 实例主机名
   eureka.instance.hostname=localhost
   # 下面两行配置表示这个是一个server端
   # 不向Eureka注册自己
   eureka.client.register-with-eureka=false
   # 不获取注册表
   eureka.client.fetch-registry=false
   # 配置默认访问地址
   eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

3. 配置注解

   ```java
   @EnableEurekaServer  // 配置在启动类上面，开启Eureka服务端配置
   @SpringBootApplication
   public class EurekaserverApplication {
   ```

#### 客户端

1. 依赖

   ```xml
   <!-- eureka 客户端依赖 -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka</artifactId>
   </dependency>
   <!-- Web项目依赖 -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

2. 配置文件

   ```properties
   # 应用名称
   spring.application.name=eureka-client
   # 服务端口
   server.port=8762
   
   # 配置默认注册地址（Eureka服务端地址）
   eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
   ```

3. 配置注解

   ```java
   @EnableEurekaClient  // 配置在启动类上面，开启Eureka客户端配置
   @SpringBootApplication
   public class ServiceHiApplication {
   ```



### Ribbon

1. 依赖

   ```xml
   <!-- Eureka -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka</artifactId>
   </dependency>
   <!-- Ribbon -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-ribbon</artifactId>
   </dependency>
   <!-- Web依赖 -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

2. 配置文件

   ```properties
   # 应用名称
   spring.application.name=eureka-client
   # 服务端口
   server.port=8762
   
   # 配置默认注册地址（Eureka服务端地址）
   eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
   ```

3. 配置注解

   ```java
   @EnableEurekaClient  // 配置在启动类上面，开启Eureka客户端配置
   @SpringBootApplication
   public class ServiceRibbonApplication {
   
   	public static void main(String[] args) {
   		SpringApplication.run(ServiceRibbonApplication.class, args);
   	}
   
       // 向Spring Ioc容器注册一个RestTemplate
   	@Bean
   	@LoadBalanced  // 通过@LoadBalanced注解，表明这个RestTemplate开启负载均衡功能，默认为轮询
   	RestTemplate restTemplate() {
   		return new RestTemplate();
   	}
   
   }
   
   
   @Service
   public class HelloService {
   
       @Autowired
       RestTemplate restTemplate;
   
       public String hiService(String name) {
           // 通过服务名访问，ribbon会根据服务名来选择具体的服务实例
           return restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
       }
   
   }
   ```



### Feign

1. 依赖

   ```xml
   <!-- Eureka -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka</artifactId>
   </dependency>
   <!-- Feign -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-feign</artifactId>
   </dependency>
   <!-- Web项目依赖 -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

2. 配置文件

   ```properties
   # 应用名称
   spring.application.name=eureka-client
   # 服务端口
   server.port=8762
   
   # 配置默认注册地址（Eureka服务端地址）
   eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
   ```

3. 配置注解

   ```java
   @EnableEurekaClient  // 配置在启动类上面，开启Eureka客户端配置
   @EnableFeignClients  // 开启Feign默认配置
   @SpringBootApplication
   public class ServiceFeignApplication {｝
       
   
   // 定义一个Feign接口，Web层的Controller层通过Feign接口来消费服务
   @FeignClient(value = "service-hi")  // 指定调用哪个服务
   public interface SchedualServiceHi {
       // 调用 service-hi 服务的 /hi 接口
       @RequestMapping(value = "/hi",method = RequestMethod.GET)
       String sayHiFromClientOne(@RequestParam(value = "name") String name);
   }
   ```



### Hystrix

1. 依赖

   ```xml
   <!-- Hystrix -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-hystrix</artifactId>
   </dependency>
   ```

2. 配置文件

   ```properties
   # 应用名称
   spring.application.name=eureka-client
   # 服务端口
   server.port=8762
   
   # 配置默认注册地址（Eureka服务端地址）
   eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
   
   # feign 中是自带断路器的，但是默认没有打开
   feign.hystrix.enabled=true
   ```

3. 配置注解（Ribbon）

   ```java
   @EnableDiscoveryClient  // 配置在启动类上面，开启Eureka客户端配置
   @EnableHystrix  // 开启Hystrix默认配置
   @SpringBootApplication
   public class ServiceRibbonApplication {}
   
   
   // 业务类
   @Service
   public class HelloService {
   
       @Autowired
       RestTemplate restTemplate;
   
       // hiService方法超时，或错误，直接调用hiError方法快速失败
       @HystrixCommand(fallbackMethod = "hiError")
       public String hiService(String name) {
           return restTemplate.getForObject("http://SERVICE-HI/hi?name="+name,String.class);
       }
   
       public String hiError(String name) {
           return "hi,"+name+",sorry,error!";
       }
   }
   ```

4. 配置注解（Feign）

   ```java
   @EnableEurekaClient  // 配置在启动类上面，开启Eureka客户端配置
   @EnableFeignClients  // 开启Feign默认配置
   @EnableHystrix  // 开启Hystrix默认配置
   @SpringBootApplication
   public class ServiceFeignApplication {｝
       
   
   // 业务类，Feign接口，fallback 表明出现错误时，直接使用指定类中的方法快速失败
   @FeignClient(value = "service-hi",fallback = SchedualServiceHiHystric.class)
   public interface SchedualServiceHi {
       @RequestMapping(value = "/hi",method = RequestMethod.GET)
       String sayHiFromClientOne(@RequestParam(value = "name") String name);
   }
   
   // 实现Feign接口，做快速失败处理
   @Component
   public class SchedualServiceHiHystric implements SchedualServiceHi {
       @Override
       public String sayHiFromClientOne(String name) {
           return "sorry "+name;
       }
   }
   ```



### Hystrix Dashboard

1. 依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
   </dependency>
   ```

2. 配置注解

   ```java
   @EnableEurekaClient  // 配置在启动类上面，开启Eureka客户端配置
   @EnableHystrix  // 开启Hystrix默认配置
   @EnableHystrixDashboard  // 开启HystrixDashboard默认配置
   @SpringBootApplication
   public class ServiceRibbonApplication {
   ```

3. 访问视图

   ```html
   http://localhost:{server.prot}/hystrix
   
   分别输入:
   	http://localhost:{server.prot}/hystrix.stream
   	2000
   	miya（这个随意）
   ```



### Zuul

1. 依赖

   ```xml
   <!-- Eureka -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-eureka</artifactId>
   </dependency>
   <!-- Zuul -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-zuul</artifactId>
   </dependency>
   <!-- Web依赖 -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

2. 配置文件

   ```properties
   # 应用名称
   spring.application.name=service-zuul
   # 服务端口
   server.port=8769
   # 配置默认注册地址（Eureka服务端地址）
   eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
   
   # zuul 路由配置
   # /api-a/ 开头的请求，都转发给 service-ribbon 服务
   zuul.routes.api-a.path=/api-a/**
   zuul.routes.api-a.service-id=service-ribbon
   
   # /api-b/ 开头的请求，都转发给 service-feign 服务
   zuul.routes.api-b.path=/api-b/**
   zuul.routes.api-b.service-id=service-feign
   ```

3. 配置注解

   ```java
   @EnableZuulProxy  // 开启Zuul默认配置
   @EnableEurekaClient  // // 开启Eureka客户端默认配置
   @SpringBootApplication
   public class ServiceZuulApplication {
   ```



### Config

#### 服务端

1. 依赖

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-config-server</artifactId>
   </dependency>
   ```

2. 配置文件

   ```properties
   # 服务名称
   spring.application.name=config-server
   # 服务端口
   server.port=8888
   # eureka 注册到Eureka
   eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
   
   # git仓库地址
   spring.cloud.config.server.git.uri=https://github.com/dengweiqiang/SpringCloudConfig/
   # 仓库路径
   spring.cloud.config.server.git.search-paths=repository
   # 仓库分支
   spring.cloud.config.label=master
   # 公有仓库不需要填写，私有仓库需要填写
   spring.cloud.config.server.git.username=your username
   spring.cloud.config.server.git.password=your password
   ```

3. 配置注解

   ```java
   @EnableConfigServer
   @SpringBootApplication
   public class ConfigServerApplication {
   ```

4. Http请求地址和资源文件映射如下：

   ```properties
   # application 应用名称
   # profile 开发环境
   # label 分支
   # 例如： zuul-dev.properties 文件，对应的访问Uri为: /zuul/dev
   /{application}/{profile}[/{label}]
   /{application}-{profile}.yml
   /{label}/{application}-{profile}.yml
   /{application}-{profile}.properties
   /{label}/{application}-{profile}.properties
   ```

#### 客户端

1. 依赖

   ```xml
   <!-- Config -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-config</artifactId>
   </dependency>
   <!-- Web依赖 -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   ```

2. 配置文件

   ```properties
   # 应用名称
   spring.application.name=service-zuul
   # 服务端口
   server.port=8769
   # 配置默认注册地址（Eureka服务端地址）
   eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
   
   
   # 配置中心
   #spring.cloud.config.uri=http://CONFIG-SERVER:8888/
   spring.cloud.config.label=master
   # 激活dev环境配置，配置文件在git仓库里面
   spring.cloud.config.profile=dev
   # 是否从配置中心读取文件
   spring.cloud.config.discovery.enabled=true
   # 配置中心的ServerId，即服务名
   spring.cloud.config.discovery.service-id=CONFIG-SERVER
   
   # 配置重试机制
   spring.cloud.config.retry.initial-interval=2000
   spring.cloud.config.retry.max-attempts=2000
   spring.cloud.config.retry.max-interval=2000
   spring.cloud.config.retry.multiplier=1.2
   spring.cloud.config.fail-fast=true
   ```

3. 配置注解

   ```java
   @EnableEurekaClient
   @EnableZuulProxy
   @RefreshScope	// 动态刷新配置
   @SpringBootApplication
   public class ServiceZuulApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(ServiceZuulApplication.class, args);
       }
   
       // 对于zuul的动态配置，必须主动向Ioc注册Bean，并且加上@RefreshScope
       @Bean
       @RefreshScope
       @ConfigurationProperties("zuul")
       @Primary
       public ZuulProperties zuulProperties() {
           return new ZuulProperties();
       }
   }
   ```

   