# 尝试在本地Docker环境快速运行Kafka

> https://qiita.com/turupon/items/12268ddb95ecd7b7ae07

## 整体配置图

![kafka-00.png](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F544022%2Fa98ea391-27e1-38b7-5b6f-141032ea92e5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=34ee746f2b518bc33777fc4117437f1d)

在运行用于在 Docker 容器中运行 zookeeper、broker 和 Producer/Consumer 的 cli。

![kafka-1.png](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F544022%2F351da61f-9ec2-efb6-bc18-71247fc022c5.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=a360528b8fd1974932ea8350c2d2c3fd)

## 网络定义

- 首先，定义将用于这种情况的网络。  

  当然，所有容器都在一个段中运行。

    ```
    $ docker network create iot_network
    7a00a485b89f83d664fef537f21e7df16ba34f14dc2731bc8686d2b0c55c1fe3
    ```

## docker-compose 文件

- 定义zookeeper、broker和cli容器的docker-compose.yml如下：  
  它还定义了要使用的网络（在上面创建）。

  ```
  version: "3"
  services:
    zookeeper:
      image: confluentinc/cp-zookeeper:5.5.1
      hostname: zookeeper
      container_name: zookeeper
      ports:
        - "32181:32181"
      environment:
        ZOOKEEPER_CLIENT_PORT: 32181
        ZOOKEEPER_TICK_TIME: 2000
  
    broker:
      image: confluentinc/cp-kafka:5.5.1
      hostname: broker
      container_name: broker
      depends_on:
        - zookeeper
      ports:
        - "9092:9092"
        - "29092:29092"
      environment:
        KAFKA_BROKER_ID: 1
        KAFKA_ZOOKEEPER_CONNECT: "zookeeper:32181"
        KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
        KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:29092,PLAINTEXT_HOST://localhost:9092
        KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
        CONFLUENT_SUPPORT_METRICS_ENABLE: "false"
  
    cli:
      image: confluentinc/cp-kafka:5.5.1
      hostname: cli
      container_name: cli
      depends_on:
        - broker
      entrypoint: /bin/sh
      tty: true
  
  networks:
    default:
      external:
        name: iot_network
  ```

## 创建和验证容器

- 构建并启动定义的容器。

  ```
  $ docker-compose up -d
      前略
  Creating zookeeper ... done
  Creating broker    ... done
  Creating cli       ... done
  ```

- 检查正在运行的容器。

  ```
  $ docker-compose ps
     Name                Command            State               Ports                         
  ----------------------------------------------------------------------------------------------------------
  broker      /etc/confluent/docker/run   Up          0.0.0.0:29092->29092/tcp, 0.0.0.0:9092->9092/tcp      
  cli         /bin/sh                     Up          9092/tcp                                              
  zookeeper   /etc/confluent/docker/run   Up          2181/tcp, 2888/tcp, 0.0.0.0:32181->32181/tcp, 3888/tcp
  ```

  

## 检查网络信息

- 还要检查分配给每个容器的网络信息。

  由于容器之间的通信是使用容器名称执行的，因此无需知道 IP 地址。

  ```
  $ docker network inspect iot_network
  
  [
      {
          "Name": "iot_network",
          "Id": "7a00a485b89f83d664fef537f21e7df16ba34f14dc2731bc8686d2b0c55c1fe3",
          "Created": "2021-02-03T15:58:22.6410234Z",
          "Scope": "local",
          "Driver": "bridge",
          "EnableIPv6": false,
          "IPAM": {
              "Driver": "default",
              "Options": {},
              "Config": [
                  {
                      "Subnet": "172.21.0.0/16",
                      "Gateway": "172.21.0.1"
                  }
              ]
          },
          "Internal": false,
          "Attachable": false,
          "Ingress": false,
          "ConfigFrom": {
              "Network": ""
          },
          "ConfigOnly": false,
          "Containers": {
              "192c2091a0cbad678edfd6165b8392be9e6e17b7b4eb74179303c366d6efa301": {
                  "Name": "broker",
                  "EndpointID": "ace1688ef02c686264325019143afef994078e53501586ebf7859c4a5b8b4ef9",
                  "MacAddress": "02:42:ac:15:00:03",
                  "IPv4Address": "172.21.0.3/16",
                  "IPv6Address": ""
              },
              "79634a9c332300d558235170e19f9ede4528d4566c95eceaf0adc31b1ab9e441": {
                  "Name": "zookeeper",
                  "EndpointID": "fcdcd02386f9f972d6a93e8d7b2820f3df8039f0749ff2c9981ca3e7754322f0",
                  "MacAddress": "02:42:ac:15:00:02",
                  "IPv4Address": "172.21.0.2/16",
                  "IPv6Address": ""
              },
              "dd58a95c5904c456e728e9fd881b50e89b1138a23424378cedde87d86c95bba0": {
                  "Name": "cli",
                  "EndpointID": "690e00daf32594623a0068a86fbbe5be90a53f42f1901606d59f035ff6c43b1c",
                  "MacAddress": "02:42:ac:15:00:04",
                  "IPv4Address": "172.21.0.4/16",
                  "IPv6Address": ""
              }
          },
          "Options": {},
          "Labels": {}
      }
  ]
  ```

## 创建主题Topic

创建一个主题来发送来自生产者的消息并接收来自消费者的消息。

- 连接到经纪人（broker）。

    ```
    $ docker exec -it broker /bin/bash
    root@broker:/#
    ```

- 创建一个名为“sample-topic”的主题。

    ```
    root@broker:/# kafka-topics --bootstrap-server broker:9092 --create --topic sample-topic --partitions 3 replication-factor 1
    Created topic sample-topic.
    ```

- 检查创建的主题

  ```
  root@broker:/# kafka-topics --bootstrap-server broker:9092 --describe --topic sample-topic
  Topic: sample-topic PartitionCount: 3   ReplicationFactor: 1    Configs: 
      Topic: sample-topic Partition: 0    Leader: 1   Replicas: 1 Isr: 1
      Topic: sample-topic Partition: 1    Leader: 1   Replicas: 1 Isr: 1
      Topic: sample-topic Partition: 2    Leader: 1   Replicas: 1 Isr: 1
  ```

  

## 设置Consumer消息接收

- 连接到 cli。

  ```
  $ docker exec -it cli /bin/bash
  root@cli:/#
  ```

- 将消息接收设置为消费者。

  ```
  root@cli:/# kafka-console-consumer --bootstrap-server broker:29092 --topic sample-topic --group G1 --from-beginning
  ```

  ↑ 没有显示任何提示，但我们正在等待 Producer 的消息。

## 设置Producer发送消息

- 打开另一个终端并连接到 cli 以发送消息。

  ```
  $ docker exec -it cli /bin/bash
  root@cli:/#
  ```

- 将消息传输设置为Producer。

  ```
  root@cli:/# kafka-console-producer --broker-list broker:29092 --topic sample-topic
  >
  ```

  在提示“>”之后，输入合适的消息。
  您刚刚键入的消息出现在消费者的提示中。

  ```
  root@cli:/# kafka-console-producer --broker-list broker:29092 --topic sample-topic
  >Hello World
  >Good Good Good !!!
  >
  --------------
  root@cli:/# kafka-console-consumer --bootstrap-server broker:29092 --topic sample-topic --group G1 --from-beginning
  Hello World
  Good Good Good !!!
  ```

  

至此，确认了从作为Kafka基础中的基本操作检查的Producer发送的消息，可以通过Broker被Consumer接收。