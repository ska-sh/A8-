![[中间件/dubbo/image/Pasted image 20230803160151.png]]
1. 调用过程从Proxy开始，Proxy持有一个Invoker对象，然后在触发invoke调用
2. 在invoke调用过程中，需要使用Cluster，Cluster负责容错
3. Cluster在调用之前会通过Directory获取所有可用的远程Invoker列表。由于可以调用的原创Invoker可能会很多，如果用户配置了路由规则，那么还会根据路由规则在过滤一遍Invoker。
4. 此时可能还会有很多可用的Invoker，接下来就可以通过LoadBalance方法做负载均衡，最终选择一个可用的Invoker。在调用之前会进入一个过滤器链，这个过滤器链通常处理上下文、限流、计数等。
5. 使用Client做数据传输，传输之前要做私有协议的构造，此时会用到Codec接口。构造完成后，对数据做序列化（Serialization），之后传入到服务提供者。
6. 服务提供者接收到数据，也会使用Codec处理协议投及一些半包，粘包等。处理完成之后再对整个数据做反序列化。
7. 之后Request会被分配到线程池（ThreadPool）中进行处理。
8. Server会处理这些Request，根据对用的请求查找到Exporter（它内部持有Invoker）。Invoker被用装饰器模式一层一层套了很多的Filter，因此在调用最终的实现类之前，有会经过服务提供者端的过滤器链
9. 最终，我们得到了具体接口的真实实现并调用，在原路把结果返回。