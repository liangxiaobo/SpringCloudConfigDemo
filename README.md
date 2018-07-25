高可用的配置中心加上使用Spring Cloud Bus动态刷新配置文件获取
##准备
这里需要三个工程Eureka-Server、Config-Server、Config-Client下面依次创建工程
![b1.png](https://upload-images.jianshu.io/upload_images/2151905-6f8866107a8bb478.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### eureka-server 服务注册发现
创建pom所需要起步依赖包 `spring-cloud-starter-netflix-eureka-server` 
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.maven.eureka.server</groupId>
    <artifactId>eureka-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>eureka-server</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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
工程创建完成之后在 `EurekaServerApplication`上添加注解 **@EnableEurekaServer**
如下:
``` java 
  @SpringBootApplication
  @EnableEurekaServer
  public class EurekaServerApplication {

      public static void main(String[] args) {
          SpringApplication.run(EurekaServerApplication.class, args);
      }
  }

```
application.properties 如下：
``` xml
server.port=8761
eureka.client.register-with-eureka=false 
eureka.client.fetch-registry=false
eureka.client.service-url.defaultZone=http://localhost:${server.port}/eureka/
```
其中 `eureka.client.register-with-eureka=false `, `eureka.client.fetch-registry=false`为false表示不注册自己
然后启动 http://localhost:8761
![b2.png](https://upload-images.jianshu.io/upload_images/2151905-2077a4550ffd55f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/850)

### config-server 配置中心服务端
创建pom所需的起步依赖 `spring-cloud-config-server spring-cloud-starter-netflix-eureka-server`
pom.xml

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.maven.config.server</groupId>
    <artifactId>config-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>config-server</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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
application.yml
``` yml
spring:
  cloud:
    config:
      server:
        git:
          search-paths: respo # 配置文件所属目录，配置文件不能放在仓库的根目录
          uri: http://IP/liangxiaobo/MyTestSpringcloudConfig.git
          username: *** # 你的用户名
          password: *** # 你的密码
      label: master

  application:
    name: config-server
server:
  port: 8768
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```
以上请注册这里用到了git仓库，我用的是自建的git创建服务器，并且是私有(private)项目，需要写用户名(username)和密码(password)，config-server做为Eureka的客户端需要向eureka server注册服务，service-url指向http://localhost:8761/eureka/
向git仓库上传一个配置文件 `respo/config-client-dev.yml`，配置文件不能放在根目录必须放在指定的目录中
``` yml
foo: foo version 1
name: springcloud-config-server
```
然后在启动类中加上注解`@EnableConfigServer @EnableEurekaClient` 代码如下：
``` java
@SpringBootApplication
@EnableConfigServer
@EnableEurekaClient
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

```
这时候想测试config-server的话，要依次启动 eureka-server、config-server
这时候访问 config-server http://localhost:8768/foo/dev
``` xml
<Environment>
  <name>foo</name>
  <profiles>
    <profiles>dev</profiles>
  </profiles>
  <label/>
  <version>3363ba036ef408f7b204aa487314c1fb133a1467</version>
  <state/>
  <propertySources/>
</Environment>
```
这是config-server项目启动时加入的映射
``` log
2018-07-25 11:45:58.679  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2018-07-25 11:45:58.680  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2018-07-25 11:45:58.688  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/encrypt],methods=[POST]}" onto public java.lang.String org.springframework.cloud.config.server.encryption.EncryptionController.encrypt(java.lang.String,org.springframework.http.MediaType)
2018-07-25 11:45:58.689  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/encrypt/{name}/{profiles}],methods=[POST]}" onto public java.lang.String org.springframework.cloud.config.server.encryption.EncryptionController.encrypt(java.lang.String,java.lang.String,java.lang.String,org.springframework.http.MediaType)
2018-07-25 11:45:58.690  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/decrypt/{name}/{profiles}],methods=[POST]}" onto public java.lang.String org.springframework.cloud.config.server.encryption.EncryptionController.decrypt(java.lang.String,java.lang.String,java.lang.String,org.springframework.http.MediaType)
2018-07-25 11:45:58.690  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/decrypt],methods=[POST]}" onto public java.lang.String org.springframework.cloud.config.server.encryption.EncryptionController.decrypt(java.lang.String,org.springframework.http.MediaType)
2018-07-25 11:45:58.690  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/encrypt/status],methods=[GET]}" onto public java.util.Map<java.lang.String, java.lang.Object> org.springframework.cloud.config.server.encryption.EncryptionController.status()
2018-07-25 11:45:58.690  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/key],methods=[GET]}" onto public java.lang.String org.springframework.cloud.config.server.encryption.EncryptionController.getPublicKey()
2018-07-25 11:45:58.691  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/key/{name}/{profiles}],methods=[GET]}" onto public java.lang.String org.springframework.cloud.config.server.encryption.EncryptionController.getPublicKey(java.lang.String,java.lang.String)
2018-07-25 11:45:58.695  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{name}-{profiles}.properties],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.properties(java.lang.String,java.lang.String,boolean) throws java.io.IOException
2018-07-25 11:45:58.696  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{name}-{profiles}.yml || /{name}-{profiles}.yaml],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.yaml(java.lang.String,java.lang.String,boolean) throws java.lang.Exception
2018-07-25 11:45:58.696  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{name}/{profiles:.*[^-].*}],methods=[GET]}" onto public org.springframework.cloud.config.environment.Environment org.springframework.cloud.config.server.environment.EnvironmentController.defaultLabel(java.lang.String,java.lang.String)
2018-07-25 11:45:58.696  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{label}/{name}-{profiles}.json],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.labelledJsonProperties(java.lang.String,java.lang.String,java.lang.String,boolean) throws java.lang.Exception
2018-07-25 11:45:58.696  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{label}/{name}-{profiles}.properties],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.labelledProperties(java.lang.String,java.lang.String,java.lang.String,boolean) throws java.io.IOException
2018-07-25 11:45:58.696  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{name}-{profiles}.json],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.jsonProperties(java.lang.String,java.lang.String,boolean) throws java.lang.Exception
2018-07-25 11:45:58.697  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{name}/{profiles}/{label:.*}],methods=[GET]}" onto public org.springframework.cloud.config.environment.Environment org.springframework.cloud.config.server.environment.EnvironmentController.labelled(java.lang.String,java.lang.String,java.lang.String)
2018-07-25 11:45:58.697  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{label}/{name}-{profiles}.yml || /{label}/{name}-{profiles}.yaml],methods=[GET]}" onto public org.springframework.http.ResponseEntity<java.lang.String> org.springframework.cloud.config.server.environment.EnvironmentController.labelledYaml(java.lang.String,java.lang.String,java.lang.String,boolean) throws java.lang.Exception
2018-07-25 11:45:58.700  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{name}/{profile}/{label}/**],methods=[GET],produces=[application/octet-stream]}" onto public synchronized byte[] org.springframework.cloud.config.server.resource.ResourceController.binary(java.lang.String,java.lang.String,java.lang.String,javax.servlet.http.HttpServletRequest) throws java.io.IOException
2018-07-25 11:45:58.701  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{name}/{profile}/{label}/**],methods=[GET]}" onto public java.lang.String org.springframework.cloud.config.server.resource.ResourceController.retrieve(java.lang.String,java.lang.String,java.lang.String,javax.servlet.http.HttpServletRequest,boolean) throws java.io.IOException
2018-07-25 11:45:58.701  INFO 17912 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/{name}/{profile}/**],methods=[GET],params=[useDefaultLabel]}" onto public java.lang.String org.springframework.cloud.config.server.resource.ResourceController.retrieve(java.lang.String,java.lang.String,javax.servlet.http.HttpServletRequest,boolean) throws java.io.IOException
2018-07-25 11:45:58.722  INFO 17912 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2018-07-25 11:45:58.722  INFO 17912 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
```
### config-client
创建pom所需要的起步依赖 `spring-boot-starter-web spring-cloud-starter-config spring-cloud-starter-netflix-eureka-server`
pom.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.maven.config.client</groupId>
    <artifactId>config-client</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>config-client</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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
启动类中加注释 `@RestController @EnableEurekaClient`，并在其中增加两个方法读取配置文件中的属性 `foo`和`name`
``` java
@SpringBootApplication
@RestController
@EnableEurekaClient
public class ConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }

    @Value("${foo}")
    String foo;

    @Value("${name}")
    String name;

    @RequestMapping("/foo")
    public String hi() {
        return foo;
    }

    @RequestMapping("/name")
    public String getName() {
        return name;
    }
}

```
在config-client 中增加一个bootstrap.yml，它先于application.yml加载
bootstrap.yml，配置获取application.yml配置文件
``` yml
spring:
  application:
    name: config-client
  cloud:
    config:
      fail-fast: true 
      discovery:
        enabled: true
        service-id: config-server
  profiles:
    active: dev

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8762
```
这里的service-id指向**config-server**项目，config-server集群用service-id起到负载均衡作用，
在获取配置文件 的时候取文件名的方式是 `${spring.application.name}-${profiles.active}` 所以这里的文件名是config-client-dev，刚才上传了一个config-client-dev.yml文件到git仓库；
下面启动测试config-client，必须依次启动eureka-server、config-server、config-client，在启动config-client前必须确保config-server已经成功注册到eureka-server
启动成功会在控制台看到
```
 c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://DESKTOP-L20CIGM:8768/
```
8768端口是config-server服务器
下面测试访问http://localhost:8762/foo http://localhost:8762/name
![b3.png](https://upload-images.jianshu.io/upload_images/2151905-e189ed218895538d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![b4.png](https://upload-images.jianshu.io/upload_images/2151905-4285ea36901ae3d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面我们再启动一台config-server服务，将端口改为8769 启动
首先看如何一个工程启动多个实例，请看这篇文章:[https://blog.csdn.net/forezp/article/details/76408139](https://blog.csdn.net/forezp/article/details/76408139)


``` yml
server:
  port: 8769
```
访问http://localhost:8761可以看到config-server有两个分别是8768和8769
![b7.png](https://upload-images.jianshu.io/upload_images/2151905-61f9b6e2979fd2dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/850)

此时，多次启动config-client会在控制台看到
```
 c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://DESKTOP-L20CIGM:8768/
```
或
```
 c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://DESKTOP-L20CIGM:8769/
```
## 接下来改造工程 加入 spring cloud bus
这里只需要改造 config-client就行，首先在 pom中增加依赖 

``` xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>
```
**这里用到了RabbitMQ，所以要先安装RabbitMQ，请自行安装**
然后在启动类中加上注释 `@RefreshScope`
``` java
@SpringBootApplication
@RestController
@EnableEurekaClient
@RefreshScope
public class ConfigClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }

    @Value("${foo}")
    String foo;

    @Value("${name}")
    String name;

    @RequestMapping("/foo")
    public String hi() {
        return foo;
    }

    @RequestMapping("/name")
    public String getName() {
        return name;
    }
}
```
然后在application.yml增加mq配置，**请注意这是springboot2.0 配置**
``` yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: test
    password: root

management:
  endpoints:
    web:
      exposure:
        include: bus-refresh
```
配置完成，启动测试，将config-client 端口改为8763再启动一台服务，
启动完成后，有两8762和8763两台config-client服务，首先将git上的配置文件内容改为
``` yml
foo: foo version 2
name: springcloud-config-server2
```
访问http://localhost:8762/foo http://localhost:8763/foo发现还是 foo version 1
现在需要我们执行post请求 http://localhost:8762/actuator/bus-refresh，用postman 发送post请求成功后

再次访问http://localhost:8762/foo http://localhost:8763/foo
发现两个config-client都是最新的内容
![b8.png](https://upload-images.jianshu.io/upload_images/2151905-2b6733aafcb26e72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![b9.png](https://upload-images.jianshu.io/upload_images/2151905-29e1ae5760e4bc0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
为什么只刷新了一个config-client而另一个也获取最新的呢，由于使用了spring cloud bus , 其它实例也会收到刷新配置的消息，也可以用destination参数指向刷新的服务名 "actuator/bus-refresh?destination=config-client:**",config-client的所有实例。
