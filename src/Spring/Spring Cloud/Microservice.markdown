# 了解微服务
## 1. 多种多样架构风格
### 1.1 传统的单体架构
> 一个项目包含多个业务模块，打包后部署到同一机器上

![spring_cloud_1.png](spring_cloud_1.png)
#### 1.1.1 优点
* 易于测试
* 统一部署
#### 1.1.2 缺点
* 开发效率低
* 部署不够灵活
* 扩展性不强
* 稳定性不高

### 1.2 小巧简单的微服务
####  1.2.1 什么是微服务
* 一系列微小的服务
* 每个微服务跑在自己的进程里
* 每个服务为独立的业务
* 独立部署
* 分布式管理

![](spring_cloud_2.png)
#### 1.3 常用的微服务组件
* 服务注册与发现
* 服务网关
* 后端服务
* 前端服务
#### 1.4 如何治理多个微服务？
* 阿里系: Dubbo、Zookeeper、SpringMVC......
* SpringCloud: Spring Cloud NetFlix Eureka、SpringBoot...... 
## 2. Spring Cloud
> SpringCloud 是一个开发工具集，包含多个子项目，利用SpringBoot的开发便利性，对NetFlix开源组件进行分装，简化了分布式开发，
### 2.1 Eureka
> 基于NetFlix Eureka做了二次分装，由Eureka Servcer(注册中心)与Eureka Client(服务发现)组成,具有心跳检测、健康检查、负载均衡等功能
#### 2.1.1 创建一个简单的Eureka注册中心
* 初始化Eureka(注意版本)
![](spring_cloud_3.png)
![](spring_cloud_4.png)
![](spring_cloud_5.png)
![](spring_cloud_6.png)
* pom
```


```
* 配置启动类
```java
@SpringBootApplication
@EnableEurekaServer //标识是Eureka Server
public class StudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(StudyApplication.class, args);
    }

}
```
* 配置properties后启动访问http://localhost:8761进行测试
```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
    # 在注册中心不显示当前应用
    register-with-eureka: false
 # 设置application 名字
spring:
  application:
    name: eureka
# 设置端口
server:
  port: 8761
```
#### 2.1.2 创建一个简单的Eureka Client端
* 与注册中心创建方式大致相同，只列出不同点，注意Spring Boot版本、依赖的版本与注册中心的保持一致
* 创建Client
![](spring_cloud_7.png)
* pom
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.xuesong</groupId>
    <artifactId>study</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>study</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

```
* 配置启动类
```java
@SpringBootApplication
@EnableEurekaClient
public class StudyApplication {

    public static void main(String[] args) {
        SpringApplication.run(StudyApplication.class, args);
    }
}
```
* 配置properties
```
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
spring:
  application:
    name: client
```
* 访问注册中心查看是否已经注册(注册中心要以后台进程的方式运行)

![](spring_cloud_8.png)
#### 2.1.3 创建高可用的Eureka注册中心
* 根据之前的方式创建两个Eureka，Eureka的端口个各不相同，application.yml中的defaultZone填写对方的地址已达到相互注册的功能
* 在Eureka Client上修改application.yml，defaultZone上填写两个Eureka注册中心的地址以 , 分割

## 3. Spring Cloud 服务间的通信
> Spring Cloud与Dubbo不同，Dubbo采用RPC方式，而SpringCloud采用HTTP
### 3.1 Spring Cloud中两种restful通信方式
#### 3.1.1 RestTemplate
* 直接使用RestTemplate调用，简单粗暴
```
RestTemplate restTemplate = new RestTemplate();
String response = restTemplate.getForObject("http://xxxx:8080/xxx",String.class);
```
* 利用loadBalancerClient根据注册名获取地址，灵活多变
```
RestTemplate restTemplate = new restTemplate();
ServiceInstance serviceInstance = loadBalancerClient.choose("注册中心的服务名称");
String url = String.format("http://%s:%s",serviceInstance.getHost(),serviceInstance.getPort())+"/xxx";
String response = restTemplate.getForObject('http://xxxx:8080/xxx',String.class);
```
* 利用@LoadBalanced注解
```
// 创建工具类
@Component
public class RestTemplateConfig{
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    } 
}
```
```
// 调用
......
@Autowired
private RestTemplate restTemplate;
public String test(){
    String response = restTemplate.getForObject("http://注册中心的服务名称/xxx",String.class);
    return response;
}
......
```
#### 3.1.2 Feign
* 在服务调用方添加Feign依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
    <scope>test</scope>
</dependency>
```
* 在启动类上添加注解 @EnableFeignClients
```
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class ProductApplication {
	public static void main(String[] args) {
		SpringApplication.run(ProductApplication.class, args);
	}
}
```
* 定义需要访问的接口
```java
@FeignClient(name="访问的服务名称")
public interface ProductClient{
    @GetMapping("/访问的接口名称")
    String getMag();
}
```
* 调用测试
```
......
@Autowired
private ProductClient productClient;
public String test(){
    String response = productClient.getMag();
    return response;
}
......
```
#### 负载据衡器Ribbon
> 使用RestTemplate、Feign、Zuul向某个Eureka发起请求时Spring Cloud会转换成Ribbon并执行相应的负载均衡策略
##### 核心功能
* 服务发现
> 根据服务名称获取该服务的所有实例
* 服务选择
> 根据策略选取有效的服务
* 服务监听
> 删除异常的实例
### 分布式
> 利用物理架构，由多个自治元素，不共享内存，但通过网路发送消息合作


## 参考
1. https://coding.imooc.com/class/chapter/187.html#Anchor





