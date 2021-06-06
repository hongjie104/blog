---
layout:     post
title:      "docker安装并启动kafka"
subtitle:   ""
date:       2021-06-06 10:27:36
catalog:    false
tags:
    - kafka
---

## 服务器是Ubuntu系统,使用以下命令安装并启动kafka

```
sudo curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## 查看版本号

执行以下命令查看容器中的kafka版本号：

```
docker exec kafka-docker_kafka_1 find / -name \*kafka_\* | head -1 | grep -o '\kafka[^\n]*'
```

执行以下命令查看zookeeper版本：

```
docker exec kafka-docker_zookeeper_1 pwd
```

## 扩展broker

在docker-compose.yml所在的文件夹下，执行以下命令即可将borker总数从1个扩展到4个

```
docker-compose scale kafka=4
```

执行命令docker ps，可见kafka容器已经扩展到4个

## 创建topic

创建一个topic，名为topic001，4个partition，副本因子2，执行以下命令即可

```
docker exec kafka-docker_kafka_1 \
kafka-topics.sh \
--create --topic topic001 \
--partitions 4 \
--zookeeper zookeeper:2181 \
--replication-factor 2
```

执行以下命令查看刚刚创建的topic，这次在容器kafka-docker_kafka_3上执行命令试试：

```
docker exec kafka-docker_kafka_3 \
kafka-topics.sh --list \
--zookeeper zookeeper:2181 \
topic001
```

查看刚刚创建的topic的情况，borker和副本情况一目了然，如下

```
docker exec kafka-docker_kafka_3 \
kafka-topics.sh \
--describe \
--topic topic001 \
--zookeeper zookeeper:2181
```

## 消费消息

执行如下命令，即可进入等待topic为topic001消息的状态

```
docker exec kafka-docker_kafka_2 \
kafka-console-consumer.sh \
--topic topic001 \
--bootstrap-server kafka-docker_kafka_1:9092,kafka-docker_kafka_2:9092,kafka-docker_kafka_3:9092,kafka-docker_kafka_4:9092
```

## 生产消息

打开一个新的窗口，执行如下命令，进入生产消息的命令行模式，**注意不要漏掉参数"-it"**，我之前就是因为漏掉了参数"-it"，导致生产的消息时虽然不提示异常，但是始终无法消费到消息

```
docker exec -it kafka-docker_kafka_1 \
kafka-console-producer.sh \
--topic topic001 \
--broker-list kafka-docker_kafka_1:9092,kafka-docker_kafka_2:9092,kafka-docker_kafka_3:9092,kafka-docker_kafka_4:9092
```

现在已经进入了生产消息的命令行模式，输入一些字符串然后回车，再去消费消息的控制台窗口看看，已经有消息打印出来，说明消息的生产和消费都成功了