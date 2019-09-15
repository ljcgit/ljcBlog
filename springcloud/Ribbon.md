## 配置
#### 1.添加pom依赖
```xml
         <!--ribbon部署-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```

#### 2.添加eureka属性
```
eureka:
  client:
    register-with-eureka: false   #false表示不向注册中心注册自己
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

#### 3.启动类上加上@EnableEurekaClient注解

#### 4.给RestTemplate添加负载均衡注解（@LoadBalanced ）
```java
    @Bean
    @LoadBalanced  //开启负载均衡
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

```

#### 5.修改服务提供地址
```java
    private static final String PREFIX = "http://MICROSERVICECLOUD-DEPT";
```


## 算法
+ RoundRobinRule：轮询
+ RandomRule：随机
+ AvailabilityFilteringRule：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问
+ WeightedResponseTimeRule：根据平均响应时间计算所有服务的权重，响应时间越快服务权重越大被选中的概率越高，刚启动时如果统计信息不足，则使用RoundRobinRule策略，等统计信息足够，会切换到WeightedResponseTimeRule
+ RetryRule：先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用服务
+ BestAvailableRule：会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务
+ ZoneAvoidanceRule：符合判断Server所在区域的性能和Server的可用性选择服务器

### 修改使用的算法
```java
@Configuration
public class ConfigBean {

    @Bean
    @LoadBalanced  //开启负载均衡
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

    @Bean
    public IRule myRule(){
        //采用随机算法替换轮询算法
        return new RandomRule();
    }

}
```
### 自定义配置类
+ 在启动类上加上@RibbonClient注解
```java
@EnableEurekaClient
@SpringBootApplication
//在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而使配置生效
@RibbonClient(name = "MICROSERVICECLOUD-DEPT",configuration = MySelfRule.class)
public class DeptConsumer80_App {

    public static void main(String[] args) {
        SpringApplication.run(DeptConsumer80_App.class,args);
    }
}
```
> 注意：MySelfRule这个自定义配置类不能放在@ComponentScan所扫描的当前包下以及子包下，否则我们自定义的这个配置类就会被所有的Ribbon客户端所共享，也就是达不到特殊化定制的目的。

**MySelfRule.class**
```java
@Configuration
public class MySelfRule {

    @Bean
    public IRule myRule(){
        return  new RandomRule();
    }

}
```

### RandomRule 源码分析
```java
import com.netflix.client.config.IClientConfig;

import java.util.List;
import java.util.concurrent.ThreadLocalRandom;

/**
 * A loadbalacing strategy that randomly distributes traffic amongst existing
 * servers.
 * 
 * @author stonse
 * 
 */
public class RandomRule extends AbstractLoadBalancerRule {

    /**
     * Randomly choose from all living servers
     */
    @edu.umd.cs.findbugs.annotations.SuppressWarnings(value = "RCN_REDUNDANT_NULLCHECK_OF_NULL_VALUE")
    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {      //选择的负载均衡算法
            return null;
        }
        Server server = null;

        while (server == null) {
            if (Thread.interrupted()) {      //线程是否被中断
                return null;
            }
            List<Server> upList = lb.getReachableServers();   //获取可用的服务列表
            List<Server> allList = lb.getAllServers();                 //获取总服务列表      

            int serverCount = allList.size();
            if (serverCount == 0) {
                /*
                 * No servers. End regardless of pass, because subsequent passes
                 * only get more restrictive.
                 */
                return null;
            }

            int index = chooseRandomInt(serverCount);
            server = upList.get(index);

            if (server == null) {
                /*
                 * The only time this should happen is if the server list were
                 * somehow trimmed. This is a transient condition. Retry after
                 * yielding.
                 */
                Thread.yield();
                continue;
            }

            if (server.isAlive()) {
                return (server);
            }

            // Shouldn't actually happen.. but must be transient or a bug.
            server = null;
            Thread.yield();   //线程让步
        }

        return server;

    }

    protected int chooseRandomInt(int serverCount) {
        return ThreadLocalRandom.current().nextInt(serverCount);
    }

	@Override
	public Server choose(Object key) {
		return choose(getLoadBalancer(), key);
	}
}
```
#### 问题需求
依旧轮询策略，但是加上新需求，每个服务器要求被调用5次。
```java
public class RandomRule_LJC extends AbstractLoadBalancerRule {

    private int total = 0;
    private int currentIndex = 0;

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;

        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            List<Server> upList = lb.getReachableServers();
            List<Server> allList = lb.getAllServers();

            int serverCount = allList.size();
            if (serverCount == 0) {
                /*
                 * No servers. End regardless of pass, because subsequent passes
                 * only get more restrictive.
                 */
                return null;
            }

            if(total < 5){
                total++;
            }else{
                total = 0;
                currentIndex = (currentIndex+1)%serverCount;
            }
           server = upList.get(currentIndex);

            if (server == null) {
                /*
                 * The only time this should happen is if the server list were
                 * somehow trimmed. This is a transient condition. Retry after
                 * yielding.
                 */
                Thread.yield();
                continue;
            }

            if (server.isAlive()) {
                return (server);
            }

            // Shouldn't actually happen.. but must be transient or a bug.
            server = null;
            Thread.yield();
        }

        return server;

    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig iClientConfig) {

    }
}
```

## 使用属性自定义Ribbon配置
#### 示例：
```
MICROSERVICECLOUD-DEPT:
  ribbon:
    NFLoadBalancerRuleClassName: com.rules.RandomRule_LJC
```

+ NFLoadBalancerClassName：配置ILoadBalancer的实现类
+ NFLoadBalancerRuleClassName：配置IRule的实现类
+ NFLoadBalancerPingClassName：配置IPing的实现类
+ NIWSServerListClassName：配置ServerList的实现类
+ NIWSServerListFilterClassName：配置ServerListFilter的实现类
