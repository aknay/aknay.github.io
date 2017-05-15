---
layout: post
title:  "Sending Messages Across the Network With Producer and Consumer Using Kafka and Zookeeper"
date:   2017-05-15 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---

In this tutorial, we will send messages across the network instead of using a local machine. We will use Python language to send messages. So make sure we have two machines or virtual machines to play with. 

<!-- excerpt -->



### Prerequisites

Please follow my tutorial post [here]({% post_url 2017-05-11-how-to-install-zookeeper-and-kafka-in-raspberry-pi-3 %}) to install Kafka and Zookeeper. This is applicable for both Raspberry Pi or any Linux system. 
Once Kafka and Zookeeper are installed, please make sure the topic `hello-kafka` is created. We will use this topic again.

### Network Configuration

Name | Installed Services | IP Address
--- | --- | ---
PC1 | Zookeeper and Kafka | 192.168.1.102 
PC2 | Producer and Consumer | 192.168.1.104

Here `Name` in the table is just for the sake of convenience and we will only use it to refer to a PC. There is no config for `Name` actually. So according to the table, `PC1` will be installed with `Zookeeper and Kafka` and Python programs (Producer and Consumer) will be run from `PC2`.
The IP address of `PC1` is important and it will be used for sending messages using that IP address.

### Kafka Config
We will modify `server.properties` at `PC1`. 

```sh
$ sudo nano /usr/local/kafka/config/server.properties
```
Then add this line.
```sh
advertised.listeners=PLAINTEXT://192.168.1.102:9092
```

### Install kafka-python
We just install by 
```sh
$ pip install kafka-python
```

### Producer
We will create a producer

```sh
$ cd ~/
$ mkdir kafka_ws
$ cd kafka_ws
$ nano producer.py
```

Then we add these lines and save it.

```py
from kafka import KafkaProducer
producer = KafkaProducer(bootstrap_servers='192.168.1.102:9092')
for _ in range(100):
     producer.send('hello-kafka', b'Sending messages using Python')
```
> Note: In this line `bootstrap_servers='192.168.1.102:9092'`, it is `PC1`'s' IP address and port which we already configured at Kafka server. 

> Note: In this line `producer.send('hello-kafka', b'Sending messages using Python')`, `hello-kafka` is the topic and `Sending messages using Python` is the message.

### Consumer
We will create a consumer as well.

```sh
$ cd ~/kafka_ws
$ nano consumer.py
```

Then add these lines and save it.

```py
from kafka import KafkaConsumer

consumer = KafkaConsumer('hello-kafka', bootstrap_servers='192.168.1.102:9092')
for msg in consumer:
        print(msg)
```

### Sending message
We start the consumer first.
```sh
$ python consumer.py 
```

Now, the consumer will wait for the message from the publisher.

Now we run the producer at another terminal.
```sh
$ python producer.py
```
You will receive this kind of messages at consumer terminal.

```
ConsumerRecord(topic=u'hello-kafka', partition=0, offset=1999, timestamp=1494736810704L, timestamp_type=0, key=None, value='Sending messages using Python', checksum=-849613570, serialized_key_size=-1, serialized_value_size=29)
```

> Note: Depending on Python (2 or 3) that you had, you need to use `$ python3 <your_program.py>` or `$ python <your_program.py>`