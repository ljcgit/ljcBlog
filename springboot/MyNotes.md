## 1.Servlet 
#### 使用@ServletComponentScan+@WebServlet注解实现
+ 定义一个类用@WebServlet(urlPatterns = "url路径")注解修饰；
urlPatterns表示映射的url路径
+ 新创建的类继承HttpServlet类，可以重写里面的doGet等方法；
+ @ServletComponentScan(basePackages = "包路径")使用该注解来进行组件扫描。
> 注解扫描类似与SpringMVC里面的<context:component-scan>标签。

> 在SpringBootApplication上使用@ServletComponentScan注解后，Servlet、Filter、Listener可以直接通过@WebServlet、@WebFilter、@WebListener注解自动注册，无需其他代码。

## 2.异步处理
### 2.1 AsyncContext 
+   AsyncContext asyncContext = req.startAsync();  
+ asyncContext.start() 来开启异步事件；
+ 必须要在@WebServlet或@WebFilter上加入asyncSupported=true，@WebServlet默认异步是不支持的；
+ 对客户端的响应将暂缓至调用AsyncContext的complete()或dispatch()方法为止（也就是必须等待调用这两个方法结束才能停止客户端响应）。

### 2.2 @EnableAsync+@Async
```java
@Service
public class AsyncService {

    //告诉spring 这是一个异步的
    @Async
    public void hello()  {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("处理数据中！");
    }
}
```

## 3.切换Servlet容器
+ 首先把在spring-boot-starter-web中tomcat给排除掉；
+ 在外部添加相应的依赖。

## 4.Spring模式注解装配
定义：一种用于声明在应用在扮演“组件”角色的注解
举例：@Component，@Service，@Configuration
装配：<context:component-scan>或@ComponentScan

##  5.SpringBoot自动扫描包
@SpringBootApplication   >  @EnableAutoConfiguration   >  @AutoConfigurationPackage  >   @Import({Registrar.class})

> 在Registrac类下有一个方法：
        public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
            AutoConfigurationPackages.register(registry, (new AutoConfigurationPackages.PackageImport(metadata)).getPackageName());
        }
该方法中指定了自动扫描的包默认就是当前@SpringBootApplication注解所使用的类包。

## 6.配置文件值自动注入
+ 首先创建一个合法的bean类；
+ 在application.yml或application.properties文件中根据相应的属性值进行赋值；
+ 在bean类上加上@ConfigurationProperties(prefix = "person") （**默认从全局配置文件中找**）
> 告诉SpringBoot将本类中的所有属性和配置文件中相关的进行绑定 ,并且指定是在配置文件下面的哪些属性进行一一映射
+ 在@ConfigurationProperties上面加上@Component注解。
> 只有这个组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能；

只有在pom文件中添加了下面的依赖关系，才可以在yml文件中有相应的提示功能。
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```
> **在利用yml进行自动注入时实际调用的是类的无参构造函数和set方法**

## 7.利用@Value()进行属性值填充
支持字面量${}从环境变量和配置文件中获取值，#{SpEL}。
> **@Value只能用于基础类型，不能用于复杂类型。**

## 8.@PropertySource &  @ImportResource
+ @PropertySource：加载指定的配置文件
```java
@PropertySource(value={"classpath:person.properties"})
@Component 
@ConfigurationProperties(prefix = "person")
```
+ @ImportResource:导入Spring的配置文件，让配置文件中的内容生效。
> 在主类上可以加上@ImportResource(locations = {"classpath:applicationContext.xml"})来使我们的配置文件生效。

**Spring推荐使用@Bean注解来配置：**
```java
@Configuration
public class MyAppCinfig {

    @Bean
    public HelloService helloService(){
        System.out.println("通过方法配置bean");
        return new HelloService();
    }

}
```

## 9.配置文件默认值？
+ 随机数：${random.uuid}，${random.int}
+ 占位符获取之前配置的值，如果没有可以使用“：”指定默认值：${person.name:name}

## 10.Profile
#### 10.1 多个Profile文件：
配置文件名为application-{profile}.properties/yml。
默认使用application.properties文件。

application-dev.properties配置文件：server.port=8081
application-prod.properties配置文件：server.port=8082
#### 10.2 yml支持多文档块方式
```yml
server:
  port: 8080
spring:
  profiles:
    active: prod

---
server:
  port: 8081
spring:
  profiles:dev
---

server:
  port: 8082
spring:
  profiles:prod
---
```

#### 10.3激活指定profile
+ 在配置文件中指定spring.profiles.active=dev
+ 在命令行中添加：--spring.profiles.active=prod
> Edit Configurations -> Program arguments -> --spring.profiles.active=prod

> 打包成jar文件，在cmd中用命令执行：java -jar springboot2demo-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev

+ 虚拟机参数：> Edit Configurations -> VM options -> -Dspring.profiles.active=dev

## 11 获得启用的自动配置
debug = true 会在控制台输出Positive matches信息。

## 12 日志
spring框架用的是JCL，SpringBoot框架用的是SLF4J和logback。
> 在引入其他框架的时候只需将框架的日志文件排除掉。

```java
Logger logger = LoggerFactory.getLogger(getClass());
```
```java
        logger.trace("这是trace日志。。。。");
        logger.debug("这是debug日志。。。。");
        logger.info("这是info日志。。。。");
        logger.warn("这是warn日志。。。");
        logger.error("这是error日志。。。");
```
日志的级别由低到高：trace<debug<info<warn<error。
SpringBoot默认只能输出info及以上级别的信息。
> 可以在属性文件中进行输出级别的调整：logging.level.com.springboot2test=trace

+ logging.file：在指定路径下生成指定日志文件。
+ logging.path：在指定目录下生成spring.log日志文件。


## 13 静态资源文件映射
```java
        public void addResourceHandlers(ResourceHandlerRegistry registry) {
            if (!this.resourceProperties.isAddMappings()) {
                logger.debug("Default resource handling disabled");
            } else {
                Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
                CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
                if (!registry.hasMappingForPattern("/webjars/**")) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{"/webjars/**"}).addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"}).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

                String staticPathPattern = this.mvcProperties.getStaticPathPattern();
                if (!registry.hasMappingForPattern(staticPathPattern)) {
                    this.customizeResourceHandlerRegistration(registry.addResourceHandler(new String[]{staticPathPattern}).addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations())).setCachePeriod(this.getSeconds(cachePeriod)).setCacheControl(cacheControl));
                }

            }
        }
```
+ 所有/webjars/**访问路径都去classpath:/META-INF/resources/webjars/下找资源。
> webjars依赖查询网址：https://www.webjars.org/

```java
@ConfigurationProperties(
    prefix = "spring.resources",
    ignoreUnknownFields = false
)
public class ResourceProperties {
    private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
  ```
+ “/**”可以访问的路径（静态文件夹）：
> classpath:/META-INF/resources/
classpath:/resources/
classpath:/static/
classpath:/public/
/

+ 欢迎页：静态资源文件夹下的所有index.html页面，被“/**”映射。
+ 所有的**/favicon.ico都是在静态资源文件下找。
+ spring.resources.static-locations来进行资源自定义（会禁止原先的资源设置）。

## 14 Thymeleaf
```java
@ConfigurationProperties(
    prefix = "spring.thymeleaf"
)
public class ThymeleafProperties {
    private static final Charset DEFAULT_ENCODING;
    public static final String DEFAULT_PREFIX = "classpath:/templates/"; 
    public static final String DEFAULT_SUFFIX = ".html";   //后缀
```
只要把html页面放在classpath:/templates/路径下，Thymeleaf就会自动渲染。

+ 导入名称空间：<html lang="en" xmlns:th="http://www.thymeleaf.org">


## 15 JPA
+ 导入相应依赖
```xml
        <!-- jpa-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- jdbc -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>

        <!-- mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```
+ 配置属性文件
```yml
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/jpatest?charset=utf8mb4&useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true
    username: root
    password: ljcgit123
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
```
+ 创建实体类
+ 创建实体相应的Repository

> 注意点：

## 16  缓存
#### 16.1 基本使用
+ 首先添加基本依赖
```xml
        <!-- cache -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
 ```
+ 在Application类上开启缓存：
> @EnableCaching   //开启缓存
+ 在相应的方法上添加@Cacheable注解

#### 注意点：
> + 在属性中使用了@Autowired注解的类，如果通过new来生成实例的话，使用了@Autowired注解的属性会为null**（不报错）**。
> + 必须给@Cacheable注解添加cacheNames属性值。

### 16.2 属性讲解
+ cacheName/value :指定缓存组件的名字。
+ key：缓存数据使用的key。默认是使用方法参数的值。
+ keyGenerator：key的生成器；可以自己指定key的生成器的组件id。（key/keyGenerator：两选一使用）。
+ cacheManager：指定缓存管理器。
+ cachResolver指定获取解析器。
+ condition：指定符合条件的情况下才缓存。
+ unless：当指定条件为true时，方法的返回值不会被缓存。
+ sync：是否使用异步模式。

### 16.3 @CachePut    修改缓存值
该注解属性和@Cacheable属性值一样，在使用时需要确保和缓存中的key一致。

### 16.4 @CacheEvict  删除缓存值
+ allEntries ：指定是否清除这个缓存中的所有数据，默认false。
+ beforeInvocation ：清除缓存是否在方法执行之前执行，默认false，表示会在该方法执行之后执行，如果出现异常就不会执行。

### 16.5 @Caching 组合注解
### 16.6 @CacheConfig 公共注解

## 17 Redis缓存
在SpringBoot中默认使用的是ConcurrentMapCacheManager。
#### 自定义RedisTemplate
```java
@Configuration
public class MyRedisConfig {

    @Bean
    public RedisTemplate<Object, User> myredisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, User> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<User> jk=new Jackson2JsonRedisSerializer<User>(User.class);
        template.setDefaultSerializer(jk);
        return template;
    }
}
```
```java
    @Autowired
    RedisTemplate<Object, User> myredisTemplate;
```
java.util.LinkedHashMap cannot be cast to com.springboot2test.springboot2demo.bean.User] with root cause

## 18 RabbitMQ
+ Message：消息，由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，包括routing-key（路由键），priority（相对于其他消息的优先权），delivery-mode（指出该消息可能需要持久性存储）等。
+ Publisher：消息的生产者，也是一个向交换器发布消息的客户端应用程序。
+ Exchange：交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
> Exchange有4种类型：direct（默认），fanout，topic，headers。
direct：单播
fanout：广播
topic：按指定样式
+ Queue:消息队列。一个消息可投入一个或多个消息队列，消息一直在队列里面，等待消费者连接到这个队列将其取走。
+ Binding：用于消息队列和交换器之间的关联。
+ Connection：网络连接。
+ Channel：信道。
+ Consummer：消息的消费者。
+ Virtual Host：虚拟主机。每个vhost本质上就是一个mini版的RabbitMQ服务器，拥有自己的队列，交换器，绑定和权限机制。默认是/。
+ Broker：消息队列服务器实体。

### 18.1配置RabbitMQ基本环境（docker）
+ 在docker中pull相应镜像：
> **docker pull registry.docker-cn.com/library/rabbitmq:3-management**
registry.docker-cn.com/library/ 是国内镜像加速地址；
3-management标签表示具有管理界面。
+ 启动镜像：
> **docker run -p 5672:5672 -p 15672:15672 --name myrabbitmq -d 24cb552c7c00**
将5672（客户端通信端口）和15672（管理端口）暴露出来，同时作为一个守护进程在后台运行。
> 访问管理页面：ip:15672  （默认账号密码都是guest）。


### 18.2利用RabbitTemplete发送消息
+ 导入ampq依赖：
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```
+ 设置配置属性值
```properties
spring.rabbitmq.host=id
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```
+ send()：需要自己构建一个Message消息体。
   convertAndSend()：只要传入一个对象，就会自动进行转换。
```java
        Map<String,Object> maps=new HashMap<>();
        maps.put("msg","这是一个消息！");
        maps.put("data", Arrays.asList("ljc",1234,true));
        rabbitTemplate.convertAndSend("exchanges.direct","springboot",maps);
```
> 在以对象为信息发送的时候，对象会被默认序列化。
+ 接收消息
```java
        Object o = rabbitTemplate.receiveAndConvert("springboot");
        System.out.println(o.getClass());
        System.out.println(o);
```
接收完消息后队列中就不存在了。
+ 设置发送转换类型（JSON）：
```java
@Configuration
public class MyAMQPConfig {

    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```

### 18.3 @EnableRabbit+@RabbitListener监听消息
```java
@Service
public class UserAmpqService {

    @RabbitListener(queues = "springboot")
    public void receive(User user){
        System.out.println(user);
    }
}
```
当消息队列中有消息进入，系统就会自动获取到消息。
或者利用Message作为参数来获取更多的消息信息。

## 19 定时任务
@EnableScheduling  +  @Scheduled
```java
@Service
public class ScheduledService {

    @Scheduled(cron = "0 * * * * MON-SAT")
    public void hello(){
        System.out.println("hello");
    }
}
```

## 20 设置过滤器
### 20.1 利用spring注解实现
+ 开启容器扫描注解：@ServletComponentScan(value="com.xssdemo.xsstest.filer")
+ 在过滤器上写上@WebFilter(urlPatterns = "/*")注解

### 20.2 提供Bean的方式
```java
@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean myFilter(){
        FilterRegistrationBean registrationBean = new FilterRegistrationBean(new XSSFilter());
        registrationBean.setName("myFilter");
        registrationBean.setOrder(1);
        return registrationBean;
    }

}
```
