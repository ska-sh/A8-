## 引入dynamic-datasource-spring-boot-starter

```xml
<!-- mybatis-plus begin -->  
<dependency>  
    <groupId>com.baomidou</groupId>  
    <artifactId>mybatis-plus-boot-starter</artifactId>  
    <version>${mybatis-plus-boot-starter.version}</version>  
</dependency>  
<dependency>  
    <groupId>com.baomidou</groupId>  
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>  
    <version>2.4.2</version>  
</dependency>
```

```properties
spring.datasource.dynamic.primary=master  
spring.datasource.dynamic.strict=false  
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource  
spring.datasource.dynamic.datasource.master.url=jdbc:mysql://172.20.10.202:3306/ai_map_dashboard?createDatabaseIfNotExist=true&zeroDateTimeBehavior=convertToNull&useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai  
spring.datasource.dynamic.datasource.master.username=app  
spring.datasource.dynamic.datasource.master.password=app@znyk  
spring.datasource.dynamic.datasource.master.driver-class-name=com.mysql.cj.jdbc.Driver  
  
spring.datasource.dynamic.datasource.dc.url=jdbc:mysql://proxy.dev:18092/ai_map_dashboard?createDatabaseIfNotExist=true&zeroDateTimeBehavior=convertToNull&useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai  
spring.datasource.dynamic.datasource.dc.username=app  
spring.datasource.dynamic.datasource.dc.password=app@znyk  
spring.datasource.dynamic.datasource.dc.driver-class-name=com.mysql.cj.jdbc.Driver
```

## 使用
在Service代码打上DS注册
```java
@Service  
@DS("dc")  
public class FactoryAndDeviceService {

}
```
## 注意
如果druid-spring-boot-starter的连接池，则需要在启动时过滤掉：spring.autoconfigure.exclude=com.alibaba.druid.spring.boot.autoconfigure.DruidDataSourceAutoConfigure
