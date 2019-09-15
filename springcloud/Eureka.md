## 1.什么是微服务？

## 2.微服务之间是如何独立通讯的？

## 3.springCloud和Dubbo有哪些区别？
| 功能点 | SpringCloud | Dubbo |  
| :-: | :-: | :-: |
| 服务注册中心 | Zookeeper、Redis | Eureka| 
| 服务调用方式 | RPC | Rest API |
| 服务网关 | Zuul | 无 | 
| 熔断器 | Hystrix | 无 |
| 配置中心 | Config | 无|
| 调用链追踪 | Sleuth | 无 |
| 消息总线 | Bus | 无 | 
| 数据流 | Stream | 无 | 
| 批量任务 | Task | 无 |

+ SpringCloud是基于RESTful api
+ Dubbo是基于RPC

## 4.SpringBoot和SpringCloud，请你谈谈对他们的理解？

## 5.什么是服务熔断？什么是服务降级？

## 6.微服务的优缺点分别是什么？说下你在项目开发中碰到的坑？

## 7.你所知道的微服务技术栈有哪些，请列举一二？

## 8.eureka和zookeeper都可以提供服务注册与发现的功能，请说说两个的区别？
> Eureka遵循AP原则。Eureka采用的是Peer to Peer对等通信。这是一种去中心化的架构，无master/slave之分，每一个Peer都是对等的。节点通过互相注册来提高可用性，每个节点需要添加一个或多个有效的serviceUrl指向其他节点。每个节点都可被视为其他节点的副本。
在集群环境中，如果某台Eureka Server宕机，Eureka Client的请求会自动切换到新的Eureka Server节点上，当宕机的服务器重新恢复后，Eureka会再次将其纳入到服务器集群管理之中。当节点开始接收客户端请求时，所有的操作都会在节点间进行复制操作，将请求复制到该Eureka Server当前所知的其它所有节点中。
节点间通过心跳契约的方式定期更新。默认情况下，如果Eureka Server在一定时间内没有接收到某个服务实例的心跳（默认周期是30秒），Eureka Server将会注销该实例（默认为90秒）。当Eureka Server节点在短时间内丢失过多的心跳时，那么这个节点就会进入自我保护模式。

