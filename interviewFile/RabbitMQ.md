## 1.rabbitmq 的使用场景有哪些？
+ 异步处理：当有几个事件不需要在乎他们的执行情况，就可以把所有的事件都提交一个队列，这样系统所花费的时间就从原来等待事件执行完成的总时间变成了将事件放到队列中所需时间。

+ 应用解耦：对于两个系统，每个系统都不需要另一个系统立马执行相应的事件并返回结果。

+ 流量消峰（秒杀）：当流量过大时，可以先把所有的信息放入到队列中，超过一定数量的信息直接丢弃掉，这样可以避免在短时间的高流量压垮应用。

## 2.要保证消息持久化成功的条件有哪些？
+ 需要在声明的+时候指定durable=True。
+ 如果exchange和queue都是持久化的，那么它们之间的binding也是持久化的，如果exchange和queue两者之间有一个持久化，一个非持久化，则不允许建立绑定。
> 注意：在创建了队列和交换机，就不能修改其标志了。


## 3.rabbitmq 有几种广播类型？
+ Direct ：类似于一种单播技术，消息必须与一个路由键完全匹配。
+ Fanout ：类似广播，消息会被发送到所有绑定的队列中。
+ Topic ：利用模式匹配来选定要被发送到的队列。(#匹配一个或多个单词，*匹配一个单词）。


## 4.rabbitmq 中 vhost 的作用是什么？
vhost是rabbitmq分配权限的最小细粒度。比如我们可以为用户分配访问哪个vhost的权限，不能为用户分配交换器或者队列的权限。
> 一个broker可以开设多个vhost，用于不同用户的权限分离。

## 5.rabbitmq 怎么避免消息丢失？
+ 通过AMPQ提供的事务机制实现；
```java
    @Test
    public void contextLoads() throws IOException, TimeoutException {
        ConnectionFactory cf = new ConnectionFactory();
        cf.setHost("203.195.133.72");
        Connection conn =  cf.newConnection();

        Channel channel = conn.createChannel();

        channel.queueDeclare("ljc.news",true,false,false,null);
        String message = String.format("时间 => %s", new Date().getTime());
        try {
            channel.txSelect(); // 声明事务
            // 发送消息
            channel.basicPublish("", "ljc.news", MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes("UTF-8"));
            channel.txCommit(); // 提交事务
        } catch (Exception e) {
            channel.txRollback();   //回滚事务
        } finally {
            channel.close();
            conn.close();
        }
     }
```
利用事务模式发送消息和非事务发送相比，所需花费要高出很多。

+ 使用发送者确认模式实现。
