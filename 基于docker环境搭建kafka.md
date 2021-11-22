##### zookeeper
```
docker run --name zookeeper -p 2181:2181 -d --network laradock_backend wurstmeister/zookeeper:latest
```

##### kafka
```
docker run -p 9092:9092 --name kafka -d -v /usr/local/kafka:/kafka -v /var/run/docker.sock:/var/run/docker.sock -v /etc/localtime:/etc/localtime --network laradock_backend -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://kafka:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka:latest

参数说明：
-e KAFKA_BROKER_ID=0  在kafka集群中，每个kafka都有一个BROKER_ID来区分自己

-e KAFKA_ZOOKEEPER_CONNECT=172.16.0.13:2181/kafka 配置zookeeper管理kafka的路径172.16.0.13:2181/kafka

-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.16.0.13:9092  把kafka的地址端口注册给zookeeper，如果是远程访问要改成外网IP,类如Java程序访问出现无法连接。

-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 配置kafka的监听端口

-v /etc/localtime:/etc/localtime 容器时间同步虚拟机的时间
```
##### 测试
```
cd /opt/kafka_2.12-2.4.1/bin/

//创建上行数据主题
kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic test-topic

//创建下行数据主题 message_command
kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic message_command

#运行kafka生产者发送消息
./kafka-console-producer.sh --broker-list localhost:9092 --topic test-topic

#发送消息
-> {"datas":[{"channel":"","metric":"temperature","producer":"ijinus","sn":"IJA0101-00002245","time":"1543207156000","value":"80"}],"ver":"1.0"}

#运行kafka消费者接收消息
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --from-beginning
```

###### 错误
```
启动docker容器时报错:
iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 5000 -j DNAT --to-destination 172.18.0.4:5000 ! -i br-ff45d935188b: iptables: No chain/target/match by that name. (exit status 1)

解决方案：重启docker
systemctl restart docker
```

##### 参考文档
- [docker安装kafka](https://www.cnblogs.com/linjiqin/p/11891776.html)
- [EMQ开启共享订阅/工作模式](https://blog.csdn.net/luo15242208310/article/details/103380864)