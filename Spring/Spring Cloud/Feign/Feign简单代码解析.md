# 一、Feign原理
![[Pasted image 20230727165558.png]]

## 1.1 将Feign接口注入到Spring容器中
```java
/**  
 * @author songkaiang  
 */@SpringBootApplication  
@EnableDiscoveryClient//向注册中心注册该服务  
@EnableFeignClients//开启feign接口扫描  
public class Application {  
  
    public static void main(String[] args) {  
        SpringApplication.run(Application.class, args);  
    }  
  
}
```
![[Pasted image 20230727170206.png]]
![[Pasted image 20230727171933.png]]
![[Pasted image 20230727171940.png]]

## 1.2 为接口的方法创建RequestTemplate
当consumer调用feign代理类时，代理类会调用SynchronousMethodHandler.invoke()创建RequestTemplate(url, 参数)
![[Pasted image 20230727175445.png]]
## 1.3 发出请求
代理类会通过RequestTemplate创建Request，然后client(URLConnect、HttpClient、OkHttp)使用Request发送请求
![[Pasted image 20230727175826.png]]

# 二、Feign参数传递的三种方式
- **restful传参** @PathVariable() 将参数id以**restful风格**拼接到路径中
```java
@RequestMapping("/getUserById/{id}") //拼接url，restful风格形式拼接传参
User getUserById(@PathVariable("id") Integer id);
```
- ?传参@RequestParam【拼接?形式的url】
```java
@RequestMapping("/deleteUserById") //拼接url， ？形式传参
User deleteUserById(@RequestParam("id") Integer id);
```
- - **pojo参数传参**  @RequestBody 将user对象以json字符串的形式传参
```java
@RequestMapping("/addUser") //将user对象以json字符串的形式传参
User addUser(@RequestBody User user);
```
# 三、Feign优化小技巧
## 3.1 feign超时
OpenFeign 底层内置了 [Ribbon] 框架，并且使用了 [Ribbon]的请求连接超时时间和请求处理超时时间作为其超时时间，而 Ribbon 默认的请求连接超时时间和请求处理超时时间都是 1s
![[Pasted image 20230727181609 1.png]]
由于1秒时间太短，我们需要手动设置超时时间，OpenFeign 的超时时间有以下两种更改方法：

1. 通过修改 Ribbon 的超时时间，被动的修改 OpenFeign 的超时时间。
2. 直接修改 OpenFeign 的超时时间(推荐使用)。

在项目配置文件 application.yml 中添加以下配置：
```yml
 
#超时优化有两种方式（默认超时时间为1000毫秒，时间太短）
#1.通过修改 Ribbon 的超时时间，被动的修改 OpenFeign 的超时时间。
ribbon:
  ConnectTimeout: 5000 #请求连接的超时时间
  ReadTimeout: 5000 #请求处理的超时时间
 
 
#开启feign日志--前提feign包下的logger日志为：debug模式
#2.直接修改 OpenFeign 的超时时间(推荐使用)
feign:
  client:
    config:
      default:
        ConnectTimeout: 5000 #请求连接的超时时间
        ReadTimeout: 5000 #请求处理的超时时间
```
## 3.3 http连接池
**专用通信组件自带连接池可以更好地对 HTTP 连接对象进行重用与管理，同时也能大大的提升 HTTP 请求的效率**

- 引入Apache HttpClient依赖
```xml
        <!--Spring Cloud OpenFeign Starter -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--http连接池-->
        <dependency>
            <groupId>io.github.openfeign</groupId>
            <artifactId>feign-httpclient</artifactId>
        </dependency>
```
- 开启Apache HttpClient使用（只要引入依赖即可，程序默认开启）
```yml
feign:
  httpclient:
    enabled: true #开启httpclient 默认开启
```
