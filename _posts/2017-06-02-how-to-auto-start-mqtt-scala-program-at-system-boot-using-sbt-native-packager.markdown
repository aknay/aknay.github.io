---
layout: post
title:  "How to Auto-start MQTT Scala Program at System Boot using SBT-Native-Packager"
date:   2017-06-02 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will use sbt-native-packager to autostart MQTT Scala program at system boot with Ubuntu OS. Then we will publish MQTT data to test whether the program is actually running in the background. Let's look more detail.

<!-- excerpt -->

### The Complete Project
You can also look the working example from my [github](https://github.com/aknay/mqtt-scala-native-sbt-packager) repo. I had tested on both Raspberry Pi 3 and Ubuntu OS.

### Prerequisites

Please follow my tutorial post [here]({% post_url 2017-05-09-how-to-install-scala-and-sbt-in-raspberry-pi-3  %}) to install Scala and SBT. This is applicable for both Raspberry Pi or any Linux system.


### Install MQTT Message Broker

```sh
$ sudo apt-get install mosquitto mosquitto-clients
```

Once the installation is done, you can start testing MQTT alone.

First, we start the subscriber.

```sh
$ mosquitto_sub -h localhost -t test
```
At another terminal, type in the following line to publish a message to the subscriber.

```sh
$ mosquitto_pub -h localhost -t test -m "hello world"
```

You should see the "hello world" message at the subscriber terminal.

### Install Systemd
Now Debian is adopting `systemd` to manage the services. So we will install systemd in the system.

```sh
$ sudo apt-get update
$ sudo apt-get install systemd
$ apt-get install systemd-sysv
```
Now `reboot` your system.
> Note: More info about systemd can be found [here](https://wiki.debian.org/systemd){:target="_blank"}.

### SBT Native Packager
At `build.sbt` add this line to support autostart.
```sh
enablePlugins(JavaServerAppPackaging, SystemdPlugin)
```
Create a file called `plugins.sbt` under `<your_project_name/project>` and add this line.
```sh
addSbtPlugin("com.typesafe.sbt" %% "sbt-native-packager" % "1.2.0-M8")
```

### MQTT
We also need to add in MQTT dependency in `build.sbt`

So final `build.sbt` file will be like this.

```
enablePlugins(JavaServerAppPackaging, SystemdPlugin)

name := "mqtt-sbt"
version := "1.0"

mainClass in Compile := Some("MainApp")

// package settings
maintainer in Linux := "aknay <aknay@outlook.com>"

packageDescription := "Mqtt SBT Message Exchanger"

libraryDependencies += "org.eclipse.paho" % "mqtt-client" % "0.4.0"

resolvers += "MQTT Repository" at "https://repo.eclipse.org/content/repositories/paho-releases/"
```


### Main Program
Create a main program called `MainApp.scala` under `<your_project_name/src/main/scala>`. Copy the following block of code and paste it. This will act like at message exchanger to test our idea.

```scala
import java.util.Calendar

import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence
import org.eclipse.paho.client.mqttv3.{IMqttDeliveryToken, MqttCallback, MqttClient, MqttMessage}

object MainApp {

  val SEND_ME = "SendMe"
  val RECEIVED_IT = "ReceivedIt"

  def main(args: Array[String]) {

    val brokerUrl = "tcp://localhost:1883"

    //Set up persistence for messages
    val persistence = new MemoryPersistence

    //Initializing Mqtt Client specifying brokerUrl, clientID and MqttClientPersistance
    val client = new MqttClient(brokerUrl, MqttClient.generateClientId, persistence)

    //Connect to MqttBroker
    client.connect

    //Subscribe to Mqtt topic
    client.subscribe(SEND_ME)

    //Callback automatically triggers as and when new message arrives on specified topic
    val callback = new MqttCallback {

      override def messageArrived(topic: String, message: MqttMessage): Unit = {
        println("Receiving Data, Topic : %s, Message : %s".format(topic, message))

        topic match {
          case SEND_ME =>
            sendTopic(client, RECEIVED_IT, "I received this: " + message.toString + " at " + Calendar.getInstance().getTime )

          case _ =>
        }
      }

      override def connectionLost(cause: Throwable): Unit = {
        println(cause)
      }

      override def deliveryComplete(token: IMqttDeliveryToken): Unit = {

      }
    }

    //Set up callback for MqttClient
    client.setCallback(callback)

  }

  def sendTopic(client: MqttClient, topic: String, msg: String): Unit = {
    val msgTopic = client.getTopic(topic)
    val messageToPublish = new MqttMessage(msg.getBytes("utf-8"))
    msgTopic.publish(messageToPublish)
    println("Publishing Data, Topic : %s, Message : %s".format(msgTopic.getName, messageToPublish))
  }

}

```

### Build SBT package
Now Go to your project folder then type the following line to build a package.

```
$ sbt debian:packageBin
```
Once the package is built, it can be found under `<your_project_name/target>` folder.
Now type the next following line to install
```sh
$ sudo dpkg -i target/mqtt-sbt_1.0_all.deb
```

Make sure there is no error during the installation.
We should check again whether the service is already running automatically.

```
$ systemctl
```
You should see on the list that `mqtt-sbt.service` is running

### Test with MQTT

Tt one terminal, type in to run the subscriber.

```
$ mosquitto_sub -h localhost -t ReceivedIt
```

And at another terminal, type in to publish a message.

```
$ mosquitto_pub -h localhost -t SendMe -m "hello"
```

You should see the following kind of message at the subscriber terminal.

```
I received this: hello at Fri Jun 02 14:37:52 CXT 2017
```

> Note: you can reboot and type in all those MQTT command lines again. It should still work.
> You can also remove the install package by typing `$ sudo dpkg --purge mqtt-sbt`