## 1.添加依赖
```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
```

## 2.配置yml属性
```yml
server:
  port: 9527

spring:
  application:
    name: microservicecloud-zuul-gateway
eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka,http://eureka7003.com:7003/eureka
  instance:
    instance-id: gateway-9527.com
    prefer-ip-address: true

info:
  app.name: ljc-microcloud
  company.name: www.ljc.com
  build.artifactId: $project.artifactId$
  build.version: $project.version$
```

## 3.在主启动类上添加@EnableZuulProxy注解

## 4.访问
> http://IP:9527/微服务名称/

## 5.服务别名
```yml
zuul:
  routes:
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
```

## 6.隐藏原服务
添加ignored-services属性
```yml
zuul:
  ignored-services: microservicecloud-dept
  routes:
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
```
> 隐藏所有的原服务：  ignored-services: "*"

## 7.设置统一前缀
```yml
zuul:
  prefix: /service
  ignored-services: "*"
  routes:
    mydept.serviceId: microservicecloud-dept
    mydept.path: /mydept/**
```

## 8.Zuul过滤器
#### 8.1 编写自定义过滤器
```java
public class PreRequestLogFilter extends ZuulFilter {

    @Override
    public String filterType() {
            return FilterConstants.PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return FilterConstants.PRE_DECORATION_FILTER_ORDER-1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        System.out.println(String.format("send %s request to %s",request.getMethod(),request.getRequestURL().toString()));
        return null;
    }
}
```

#### 8.2 在启动类中设置相应的bean
```java
    @Bean
    public PreRequestLogFilter preRequestLogFilter(){
        return new PreRequestLogFilter();
    }
```

#### 8.3 运行结果
![Zuul过滤器](https://upload-images.jianshu.io/upload_images/16503287-faba550bd862df20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 8.4 禁用过滤器
+ zuul.<SimpleClassName>.<filterType>.disable=true

**示例：**
```
zuul:
  PreRequestLogFilter:
    pre:
      disable: true
```
