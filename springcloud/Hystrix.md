## 1.服务熔断使用
```java
    @GetMapping(value="/dept/get/{id}")
    @HystrixCommand(fallbackMethod = "processHystrix_Get")
    public Dept get(@PathVariable("id")Long id){
        Dept dept = this.service.get(id);
        if(null == dept){
            throw new RuntimeException("该ID:"+id+"没有对应的信息");
        }
        return dept;
    }

    public Dept processHystrix_Get(@PathVariable("id")Long id){
        return new Dept().setDeptno(id).setDname("该ID:"+id+"没有对应的信息，null--@HystrixCommand").setDbSource("no this database in MySQL");
    }

```
> 一旦调用服务方法失败并抛出了错误信息后，会自动调用@HystrixCommand标注好的fallbackMethod调用类中的指定方法。

#### 1.1 获得进入回退方法的原因
```java
    public Dept processHystrix_Get(Long id,Throwable throwable){
        System.out.println("进入回退方法，异常:"+throwable);
        return new Dept().setDeptno(id).setDname("该ID:"+id+"没有对应的信息，null--@HystrixCommand").setDbSource("no this database in MySQL");
    }
```
> 只需要在方法上添加Throwable参数。


#### 1.2 配置不想执行回退的异常类
```java
    @HystrixCommand(fallbackMethod = "processHystrix_Get",ignoreExceptions = Exception.class)
```


## 2.出现如下错误
```xml
org.springframework.web.util.NestedServletException: Request processing failed; nested exception is com.netflix.hystrix.contrib.javanica.exception.FallbackDefinitionException: fallback method wasn't found: processHystrix_Get([class java.lang.Long])
	
```
> 是因为原方法和备用方法的参数个数或类型不一致造成的。

## 3.服务雪崩
多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其他的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务的调用响应就会占用越来越多的系统资源，进而引起系统崩溃，所谓的"雪崩效应”。

## 4.服务熔断
> 一般是某个服务故障或者异常引起，类似现实世界中的“保险丝”，当某个异常条件被触发，直接熔断整个服务，而不是一直等到此服务超时。

## 5.服务降级（客户端完成）
> 整体资源快不够了，忍痛将某些服务先关掉，待渡过难关，再开启回来。

> 服务降级处理是在客户端实现完成的，与服务端没有关系。


## 6.Hystrix Dashboard
除了隔离依赖服务的调用以外，Hystrix还提供了准实时的调用监控（Hystrix Dashboard），Hystrix会持续地记录所有通过Hystrix发起的请求的执行信息，并以统计报表和图形的形式展示给用户，包括每秒执行多少请求多少成功，多少失败等。

#### 6.1 添加依赖
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        </dependency>
```

#### 6.2 添加注解
```java
    @EnableHystrixDashboard
```

#### 6.3 连接失败原因？
+ 监控的服务需要实现Hystrix


## 7.聚合监控数据
#### 7.1 新建Turbine Module
#### 7.2 添加依赖
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-turbine</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
```

#### 7.3 添加@EnableTurbine
#### 7.4 通过IP:PORT/turbine.stream访问


