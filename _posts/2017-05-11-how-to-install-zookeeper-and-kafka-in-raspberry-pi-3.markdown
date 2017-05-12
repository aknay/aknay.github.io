---
layout: post
title:  "How to install Zookeeper and Kafka in Raspberry Pi 3"
date:   2017-05-11 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---

In this tutorial, we will install Zookeeper and Kafka in Raspberry Pi 3. It is also applicable for Linux system as well. After the installation, we will verify the setup by sending a message. Shall we start?

<!-- excerpt -->



### Related Topics

1. [Reference](https://github.com/keiraqz/RaspPiDemo/tree/master/kafka_config){:target="_blank"}
2. [Reference](https://www.tutorialspoint.com/apache_kafka/index.htm){:target="_blank"}


### Java JDK 8
Scala needs JDK 8 and it is already installed on Raspberry Pi 3. If it is not installed due to some reasons, you can install by
`$ sudo apt-get install oracle-java8-jdk`
> Note: You can check Java version with `$ java -version`



### Download Zookeeper
You can check latest version from [here](http://www-eu.apache.org/dist/zookeeper/){:target="_blank"}

First we will download and save it to `usr/local`
```sh
$ cd ~/
$ wget http://www-eu.apache.org/dist/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
$ sudo tar -zxvf zookeeper-3.4.9.tar.gz -C /usr/local
$ sudo mv /usr/local/zookeeper-3.4.9/ /usr/local/zookeeper
$ rm zookeeper-3.4.9.tar.gz
```
### Configure Zookeeper
We will configure the zookeeker file. 
> Note: You can read more config example from `/usr/lib/zookeeper/conf/zoo_sample.cfg`

```sh
$ sudo nano /usr/local/zookeeper/conf/zoo.cfg
```

Then copy the following lines and save it.

```sh
# The number of milliseconds of each tick
tickTime=5000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
dataDir=/usr/local/zookeeper
# the port at which the clients will connect
clientPort=2181

server.1=raspberrypi-1:2888:3888
```

### Start Zookeeper Server
You can just run server by
```
$ sudo /usr/local/zookeeper/bin/zkServer.sh start
```
> Note: If you want to stop the server; use this `sudo /usr/local/zookeeper/bin/zkServer.sh stop`


### Download Kafka
You can check latest version from [here](https://kafka.apache.org/downloads){:target="_blank"}

```
$ cd ~/
$ wget http://www-eu.apache.org/dist/kafka/0.10.2.1/kafka_2.12-0.10.2.1.tgz
$ sudo tar -zxvf kafka_2.12-0.10.2.1.tgz -C /usr/local/
$ sudo mv /usr/local/kafka_2.12-0.10.2.1/ /usr/local/kafka
$ rm kafka_2.12-0.10.2.1.tgz
```

### Configure Kafka Server
Due to memory limitation of Raspberry Pi, we should configure like this. We will start editing 
```sh
$ sudo nano /usr/local/kafka/bin/kafka-server-start.sh
```
Add this two line on top

```sh
export JMX_PORT=${JMX_PORT:-9999}
export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
```

Now we edit one more file.

```sh
sudo nano /usr/local/kafka/bin/kafka-run-class.sh
```

Replace the following block of code
```sh
# JVM performance options
if [ -z "$KAFKA_JVM_PERFORMANCE_OPTS" ]; then
  KAFKA_JVM_PERFORMANCE_OPTS="-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+DisableExplicitGC -Djava.awt.headless=true"
fi
```

with this block of code. Then save it.

```sh
if [ -z "$KAFKA_JVM_PERFORMANCE_OPTS" ]; then
KAFKA_JVM_PERFORMANCE_OPTS="-client -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+CMSScavengeBeforeRemark -XX:+DisableExplicitGC -Djava.awt.headless=true"
fi
```
### Run Kafka Server
You can just run it by typing the following line. Make sure Zookeeper is running as well.
```sh
sudo /usr/local/kafka/bin/kafka-server-start.sh /usr/local/kafka/config/server.properties &
```

### Verification
Let's verify Zookeeper and Kafka together. Please make sure ZooKeeper and Kafka servers are running.

First we will create a topic `hello-kafka`
```sh
/usr/local/kafka/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic hello-kafka
```

> Note: You can list all topics by `/usr/local/kafka/bin/kafka-topics.sh --list --zookeeper localhost:2181`

Let's start the producer console first.
```sh
/usr/local/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic hello-kafka
```

Let's start consumer console in another terminal.
```sh
/usr/local/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic hello-kafka
```

Now if you type in something at terminal of Producer console and enter, it will be displayed in Consumer console as well. That's it.