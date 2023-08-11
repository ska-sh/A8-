## 前言
通过手写SpringBoot框架，来熟悉SpringBoot大概是如何工作的
分为以下4个方面
1. 手写模拟SpringBoot启动过程
2. 手写模拟SpringBoot条件注入功能
3. 手写模拟SpringBoot自动配置功能
4. SpringBoot整合Tomcat底层源码分析

## 实战
### 依赖
首先，SpringBoot是基于Spring，所以要依赖Spring，然后我门希望模拟出来的SpringBoot也支持Spring MVC的那一套功能，所以也要依赖Spring MVC，包括Tomcat等，所以在SpringBoot中需要添加以下依赖：
```xml
<dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.3.18</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.3.18</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.18</version>
        </dependency>
 
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
        </dependency>
 
        <dependency>
            <groupId>org.apache.tomcat.embed</groupId>
            <artifactId>tomcat-embed-core</artifactId>
            <version>9.0.60</version>
        </dependency>
    </dependencies>
```
### 核心注解和核心类
在真正使用SpringBoot时，核心会用到SpringBoot一个类和注解：
1. @SpringBootApplication，这个注解是加载应用启动类上的，就是main方法所在类
2. SpringApplication，这个类中有个run()方法，用来启动SpringBoot应用

一个@SkaSpringBootApplication注解：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Configuration
@ComponentScan
public @interface SkaSpringBootApplication {
}
```

一个用来实现启动逻辑的SkaSpringApplication类
```java
public class SkaSpringApplication {
 
    public static void run(Class clazz){
 
    }
 
}
```

有了以上两者，我们就可以在MyApplication中来使用了，比如：
```java
@SkaSpringBootApplication
public class MyApplication {
 
    public static void main(String[] args) {
        GuashupringApplication.run(MyApplication.class);
    }
}
```

现在看起来有模有样了， 但是中看不中用，我们接下来要实现run方法中的逻辑

### run方法
Spring MVC的底层原理有一个Servlet非常核心，那就是DispatcherServlet，这个DispatcherServer需要绑定一个Spring容器，因为DispatcherServlet接收到请求后，就会从所绑定的Spring容器中找到所匹配的Controller，并执行匹配方法。

所以在run方法中，我们需要实现一下逻辑：
1. 创建一个Spring容器
2. 创建Tomcat对象
3. 生产DispatcherServlet对应，并且和前面创建出来的Spring容器进行绑定
4. 将DispatcherServlet添加到Tomcat中
5. 启动Tomcat

创建Spring容器，代码如下：
```java
public class SkaSpringApplication {
 
    public static void run(Class clazz){
        AnnotationConfigWebApplicationContext applicationContext = new AnnotationConfigWebApplicationContext();
        applicationContext.register(clazz);
        applicationContext.refresh();
 
        
    }
}
```

我们创建的是一个AnnotationConfigWebApplicationContext容器，并把run方法传入进来的class做为一个容器配置类，比如在MyApplication中的run方法，我们就把MyApplication.class传入到run方法中，最终MyApplication就是所创建出来的Spring容器的配置类，并且由于MyApplication类上有@SkaSpringBootApplication注解，而@SkaSpringBootApplication注解的定义上又存在@ComponentScan注解，所以在AnnotationConfigWebApplicationContext容器在执行refresh时，就会解析MyApplication这个配置类，从而发现定义了@ComponentScan注解，也就知道了要进行扫描，只不过扫描路径为空，而AnnotationConfigWebApplicationContext容器会出来这种情况，如果扫描路径为空，则会将MyApplication所在的包路径作为扫描路径。

所以在Spring容器创建完成后，容器内部就拥有了所有的Bean

### 启动Tomcat
我们用的Embed-Tomcat，也就是内嵌的Tomcat，真正的SpringBoot中也用的是内嵌的Tomcat，启动Tomcat代码如下：
```java
public static void startTomcat(WebApplicationContext applicationContext){
    
    Tomcat tomcat = new Tomcat();
    
    Server server = tomcat.getServer();
    Service service = server.findService("Tomcat");
    
    Connector connector = new Connector();
    connector.setPort(8081);
    
    Engine engine = new StandardEngine();
    engine.setDefaultHost("localhost");
    
    Host host = new StandardHost();
    host.setName("localhost");
    
    String contextPath = "";
    Context context = new StandardContext();
    context.setPath(contextPath);
    context.addLifecycleListener(new Tomcat.FixContextListener());
    
    host.addChild(context);
    engine.addChild(host);
    
    service.setContainer(engine);
    service.addConnector(connector);
    
    tomcat.addServlet(contextPath, "dispatcher", new DispatcherServlet(applicationContext));
    context.addServletMappingDecoded("/*", "dispatcher");
    
    try {
        tomcat.start();
    } catch (LifecycleException e) {
        e.printStackTrace();
    }
    
}
```

代码虽然看上去很多，但是逻辑不复杂，比如配置Tomcat绑定的端口为8081，后面想Tomcat中添加DispatcherServlet，并设置一个Mapping关系，最后启动。

在构造DispatcherServlet时，传入一个ApplicationContext对象，也就是Spring容器，也是前文说的DispatcherServlet对象和一个Spring容器进行绑定。

接下来，只需要在run方法中，调用startTomcat即可：
```java
public static void run(Class clazz){
    AnnotationConfigWebApplicationContext applicationContext = new AnnotationConfigWebApplicationContext();
    applicationContext.register(clazz);
    applicationContext.refresh();
    
    startTomcat(applicationContext);
    
}
```