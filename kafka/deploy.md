# Linux服务器上docker配置Kafka环境

## 1.拉取Kafka和zookeeper
```
docker pull wurstmeister/kafka
docker pull wurstmeister/zookeeper
```

## 2.启动zookeeper
```
docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper
```

## 3.启动kafka
```
docker run --name kafka01 \
-p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=服务器IP地址:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://服务器IP地址:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
-d  wurstmeister/kafka

```
