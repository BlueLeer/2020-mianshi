# Kafka

简介

## Kafka安装

安装Kafka步骤:

1. 安装zookeeper

```shell
docker run -d --name zookeeper -p 2181:2181 -v /etc/localtime:/etc/localtime zookeeper:3.5.6
```

2. 安装kafka

```shell
docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=122.51.109.246:2181/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://122.51.109.246:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -v /etc/localtime:/etc/localtime wurstmeister/kafka:2.12-2.3.0
```

**参数解释**：

`-e KAFKA_BROKER_ID=0` ：在Kafka中集群中,每个Kafka都有一个Broker_ID来区分自己

`-e KAFKA_ZOOKEEPER_CONNECT=122.51.109.246:2181/kafka`：配置zookeeper管理kafka的路径122.51.109.246:2181/kafka

`-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://122.51.109.246:9092`：把kafka的地址端口注册给zookeeper

`-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092`：配置kafka的监听端口

`-v /etc/localtime:/etc/localtime wurstmeister/kafka`：容器时间同步docker的宿主机时间

> tips
>
> 使用了最新的zookeeper版本：wurstmeister/zookeeper:lastest 和最新的kafka版本:wurstmeister/kafka:lastest，结果在启动kafka的时候，报错了，错误是连接不上zookeeper，因此更换了上述版本

启动完成以后，登录进kafka容器，尝试发送消息

```shell
[root@VM_0_13_centos ~]# docker exec -it kafka /bin/bash
bash-4.4# cd opt
bash-4.4# ls
kafka             kafka_2.12-2.3.0  overrides
bash-4.4# cd kafka_2.12-2.3.0/
bash-4.4# ls
LICENSE    NOTICE     bin        config     libs       logs       site-docs
bash-4.4# cd bin/
bash-4.4# ls
connect-distributed.sh               kafka-consumer-perf-test.sh          kafka-reassign-partitions.sh         kafka-verifiable-producer.sh
connect-standalone.sh                kafka-delegation-tokens.sh           kafka-replica-verification.sh        trogdor.sh
kafka-acls.sh                        kafka-delete-records.sh              kafka-run-class.sh                   windows
kafka-broker-api-versions.sh         kafka-dump-log.sh                    kafka-server-start.sh                zookeeper-security-migration.sh
kafka-configs.sh                     kafka-log-dirs.sh                    kafka-server-stop.sh                 zookeeper-server-start.sh
kafka-console-consumer.sh            kafka-mirror-maker.sh                kafka-streams-application-reset.sh   zookeeper-server-stop.sh
kafka-console-producer.sh            kafka-preferred-replica-election.sh  kafka-topics.sh                      zookeeper-shell.sh
kafka-consumer-groups.sh             kafka-producer-perf-test.sh          kafka-verifiable-consumer.sh
bash-4.4# ./kafka-console-producer.sh --broker-list localhost:9092 --topic sun
>hello-lee
```

进入到`/opt/kafka_2.12-2.3.0/bin`目录下，使用了`./kafka-console-producer.sh --broker-list localhost:9092 --topic sun` 运行生产者客户端，并发送了一条消息：hello-lee

接下来打开另一个窗口，登录消费者客户端，并订阅名为`sun`的topic：

```shell
[root@VM_0_13_centos ~]# docker exec -it kafka /bin/bash
bash-4.4# cd /opt
bash-4.4# cd kafka_2.12-2.3.0/
bash-4.4# cd bin/
bash-4.4# kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic sun --from-beginning
hello-lee
```

可以看到成功的接收到了hello-lee的消息。

至此，kafka已经安装完毕。



同样的，我们可以登录zookeeper，看看里面是个啥样子的：

```shell
# 1.登录zookeeper容器
[root@VM_0_13_centos ~]# docker exec -it zookeeper /bin/bash
# 2.进入bin目录,并看看里面都有些啥
root@3f74f16b91ee:/apache-zookeeper-3.5.6-bin# cd bin
root@3f74f16b91ee:/apache-zookeeper-3.5.6-bin/bin# ls
README.txt  zkCleanup.sh  zkCli.cmd  zkCli.sh  zkEnv.cmd  zkEnv.sh  zkServer-initialize.sh  zkServer.cmd  zkServer.sh  zkTxnLogToolkit.cmd  zkTxnLogToolkit.sh

# 3.运行zkCli.sh进入zookeeper客户端
root@3f74f16b91ee:/apache-zookeeper-3.5.6-bin/bin# ./zkCli.sh
Connecting to localhost:2181
2020-02-23 04:03:30,532 [myid:] - INFO  [main:Environment@109] - Client environment:zookeeper.version=3.5.6-c11b7e26bc554b8523dc929761dd28808913f091, built on 10/08/2019 20:18 GMT
2020-02-23 04:03:30,541 [myid:] - INFO  [main:Environment@109] - Client environment:host.name=3f74f16b91ee
2020-02-23 04:03:30,541 [myid:] - INFO  [main:Environment@109] - Client environment:java.version=1.8.0_242
2020-02-23 04:03:30,545 [myid:] - INFO  [main:Environment@109] - Client environment:java.vendor=Oracle Corporation
2020-02-23 04:03:30,545 [myid:] - INFO  [main:Environment@109] - Client environment:java.home=/usr/local/openjdk-8
...(省略)

# 4.运行`ls /`命令查看zookeeper下面都挂在了那些目录
[zk: localhost:2181(CONNECTED) 5] ls /
[kafka, zookeeper]

更多详细操作，请参考：https://www.jianshu.com/p/e8c29cba9fae
```







