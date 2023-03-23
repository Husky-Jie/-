# 注册RestTemplate

若一个服务向另一个服务发出请求获得数据信息，要在该服务的启动类注册RestTemplate

```java
/**
     * 创建RestTemplate并注入Spring容器，可发起http请求
     * @LoadBalanced 负载均衡
     */
    @Bean
//    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
```



# 实现远程调用

在service中注入RestTemplate，RestTemplate会自动将json反序列化为对象

![image-20230313215822923](D:\Java学习\java笔记\springcloud\assets\image-20230313215822923.png)



# Eureka注册中心

在Eureka架构中，微服务角色有两类

- EurekaServer:服务端，注册中心
  - 记录服务信息
  - 心跳监控
- EurekaClient:客户端
  - Provider:服务提供者
    - 注册自己的信息到EurekaServer
    - 每隔30秒向EurekaServer发送心跳
  - consumer:服务消费者
    - 根据服务名称从EurekaServer拉取服务列表
    - 基于服务列表做负载均，选中一个微服务后发起远程调用



# nacos注册中心



## 1）引入依赖

在父工程的pom文件中的`<dependencyManagement>`中引入SpringCloudAlibaba的依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.6.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

然后在消费者服务和提供者服务中的pom文件中引入nacos-discovery依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

## 2）配置nacos地址

在消费者服务和提供者服务的application.yml中添加nacos地址：

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
```

