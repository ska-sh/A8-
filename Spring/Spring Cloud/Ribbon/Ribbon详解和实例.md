**Ribbon是一个为客户端提供负载均衡功能的服务，它内部提供了一个叫做ILoadBalance的接口代表负载均衡器的操作，比如有添加服务器操作、选择服务器操作、获取所有的服务器列表、获取可用的服务器列表等等。**

## Ribbon组件IRule
**默认的是RoundBobinRule（轮询）**

|策略命名|命名|说明|
|--------------|--------------|--------------|
|RandomRule|随机策略|随机选择Server|
|RoundRobinRule|轮询车略|按顺序循环选择Server|
|RetryRule|重试策略|在一个配置时间段内选择Server不成功，则一直尝试选择一个可用的Server|
|BestAvailableRule|最低并非策略|逐个考察Server 如果Server断路器打开，则忽略，在选择其中并发链路最低的Server|
|AvailabilityFilteringRule|可用过滤策略|过滤掉一直连接失败并为circuit tripped的Server,过滤掉那些高并发的连接Server(active connections超过配置的网值)|
|ResponseTimeWeightedRule|响应时间加权策略|更具Server的响应时间分配权重。响应时间越长，权重越低|
|ZoneAvoidanceRule|区域权重策略|综合判断Server所在区域的性能和Server的可用轮询选择Server,并且判断一个AWS Zone的运行性能是否可用，剔除不可用的Zone中的所有Server|


##RetryRule
1、先按照RoundRobinRule(轮询)的策略获取服务，如果获取的服务在失败侧，在指定的时间会进行重试，进行获取可用的服务
2、如多次获取某个服务失败，则不会再再次获取该服务。如（高德地图上某条道路堵车，司机不会走那条道路）

```java
@Configuration
public class ConfigBean //boot -->spring   applicationContext.xml --- @Configuration配置   ConfigBean = applicationContext.xml
{ 
	@Bean
	@LoadBalanced
	public RestTemplate getRestTemplate(){
		return new RestTemplate();
	}
	
	@Bean
	public IRule myRule(){
		//return new RoundRobinRule();
		//return new RandomRule();//达到的目的，用我们重新选择的随机算法替代默认的轮询。
		return new RetryRule();  //在这里选择负载均衡算法
	}
}
```

## Ribbon模块介绍
- ribbon:Ribbon功能应用入口。使用Ribbon的功能可以通过初始化应用入口，调用接口实现。该模块依赖其他模块实现所需功能，比如容错处理依赖Histrix。
- ribbon-loadbalancer:负载均衡功能入口。如果仅需要负载均衡功能，可以单独使用该模块。
- ribbon-eureka:基于Eureka客户端实现服务列表管理
- ribbon-transport:具备客户端负载均衡能力的，基于RxNetty框架能够支持HTTP、TCP、UDP协议的通信客户端
- ribbon-httpclient:具备客户端负载均衡能力的，基于Apache HttpClient，支持REST风格的HTTP请求客户端
- ribbon-core:客户端配置APIs和其他共享APIs
- ribbon-example: 使用例子