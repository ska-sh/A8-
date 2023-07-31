![[Pasted image 20230727144005.png]]

Spring Cloud 常见的集成方式是使用Feign+Robbon技术来完成服务间远程调用及负载均衡，如下图

![[Pasted image 20230727144444.png]]

1. 在微服务启动时，会向服务发现中心上报自身实例信息。包括：IP地址，端口信息等
2. 微服务会定期从Nacos Server（服务发现中心）获取服务实例列表
3. 当ServiceA调用ServiceB时，ribbon组件从本地服务实例列表中查找ServiceB的实例，如获取：Instance1、Instance2。这时ribbon会通过用户配置的负载均衡策略从列表中选择一个实例。
4. 最终，Feign组件会通过ribbon选取的实例发送http请求

采用Feign+Ribbon的整合方式，是由Feign完成远程调用的整个流程。而Feign集成了Ribbon，Feign使用Ribbon

完成调用实例的负载均衡。

# 一、简介
## 1.1、负载均衡的概念
在SpringCloud服务协议流程中，ServiceA通过负载均衡调用ServiceB，下面了解一下<font color="#dd0000">负载均衡</font>：

<font color="#dd0000">负载均衡</font>就是将用户请求（流量）通过一定的策略，分摊在多个服务实力上执行，它在系统处理高并非，缓解网络眼里和进行服务器端扩容的重要手段之一。它分为<font color="#dd0000">服务端负载均衡</font>和<font color="#dd0000">客户端负载均衡</font>。

<font color="#dd0000">服务器端负载均衡</font>:
![[Pasted image 20230727160358.png]]

<font color="#dd0000">客户端负载均衡</font>
![[Pasted image 20230727160528.png]]

## 2.2、Feign概念
Feign中文含义为“假装，伪装，变形“，是一个http请求调用的轻量级框架，是以java接口注解的方式调用http接口。Feign通过处理注解，将请求模版化，当实际调用的时候，传入参数，根据参数在应用到请求上，今儿个转化成真正的请求，在中请求想到而言比较直观。

# 二、入门案例
使用Feign代替RestTemplate发送Rest请求，使之更符合面向接口化的编程习惯。
**实现步骤：**
1. 导入feign依赖starter
2. 编写Feign客户端接口
3. 消费着启动引导类开启Feign功能注解
4. 访问接口测试

**实现过程：**
## 2.1、导入依赖
- 在项目中添加<font color="#dd0000">spring-cloud-starter-openfeign</font>依赖
```xml
<!--配置feign-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```
## 2.2、Feign的客户端
- 在项目中编写Feign客户端接口类ConsumerService
```java
@FeignClient(value = "provider-service")
public interface ConsumerService {
    
    //String url = String.format("http://provider-service/user/findUserById/%s",id);
    @RequestMapping("/user/findUserById/{id}")
    User findUserById(@PathVariable("id") Integer id);

}
```
- Feign会通过动态代理，帮助我们生成实现类
- 注解@FeignClient声明Feign的客户端，指明服务名称
- 接口定义的方法，采用SpringMVC的注解。Feign会根据注解帮助我们生成URL地址
## 2.3、调用Feign

- 编写ConsumerFeignController，使用ConsumerService访问
	- @Autowired注入ConsumerService
```java
@RestController
@RequestMapping("/consumer")
public class ConsumerController {

    @Autowired
    private ConsumerService consumerService;

    @GetMapping("/findUserById/{id}")
    public User findUserById(@PathVariable Integer id){
        return consumerService.findUserById(id);
    }

}
```
## 2.4、开启Feign功能
- 在ConsumerApplication启动类上，添加<font color="#dd0000">@EnableFeignClients</font>注解，开启Feign功能
```java
@SpringBootApplication
@EnableFeignClients  //开启feign
@EnableDiscoveryClient
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class,args);
    }
}
```
## 2.5、启动测试
## 2.6、Feign实现原理简单分析
![[Pasted image 20230727163005.png]]

**Feign帮我们做了哪些事：**
- 在声明Feign客户端之后，Feign会根据@FeignClient注解使用Java的动态代理技术生成代理类，我们指定@FeignClient的value为service，则说明这个类的远程目标为的微服务名为serviceB
- serviceB的具体访问地址，Feign会交由ribbon获取，若该服务有多个实例地址，ribbon会根据配置的负载均衡策略选择服务实例
- Feign兼容spring的web注解（如：@GetMapping）,它会分析声明Feign客户端中的spring注解，得出Http请求method，参数信息及返回信息结构
- 当业务调用Feign客户端方法时，会调用代理类，根据以上分析，右代理类完成实际参数分装，远程http请求，返回结果分装等操作。
# 三、负载均衡（Ribbon）
Feign本身集成了Ribbon，因此不需要额外引入依赖
![[Pasted image 20230727164048.png]]
Ribbon是一个客户端负载均衡器，它的责任是从一组实例列表中挑选合适的实例，如何挑选？取决于**负载均衡策略** 。

# 四、熔断器支持
Feign本身也集成Hystrix熔断器，starter内查看
![[Pasted image 20230727164247.png]]

**服务降级方法实现步骤：**
1. 在配置文件application.yml中开启feign熔断支持
2. 编写FallBack处理类，实现FeignClient客户端
3. 在@FeignClient注解中，指定FallBack处理类
4. 测试服务降级效果

**实现过程**
1. 在配置文件application.yml中开启feign熔断器支持：**默认关闭**
```yml
feign:
	hystrix:
		enabled: true # 开启Feign的熔断功能
```
2. 定义一个类ConsumerServiceImpl，实现刚才编写的ConsumerService，作为FallBack的处理类
```java
@Component
public class ConsumerServiceImpl implements ConsumerService {

    //熔断方法
    @Override
    public User findUserById(Integer id) {
        User user = new User();
        user.setId(id);
        user.setNote("网络异常，请稍后再试...");
        return user;
    }
}
```
3. 在@FeignClient注解中，指定FallBack处理类。。
```java
@FeignClient(value = "provider-service",fallback = ConsumerServiceImpl.class)
public interface ConsumerService {

    //String url = String.format("http://provider-service/user/findUserById/%s",id);
    @RequestMapping("/user/findUserById/{id}")
    User findUserById(@PathVariable("id") Integer id);

}
```