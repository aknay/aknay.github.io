---
layout: post
title:  "Akka UDP Messaging"
date:   2018-01-18 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will create an Akka UDP system and send a message between localhost using different ports.
<!-- excerpt -->

#### **Git Hub Project**
The complete project can be cloned from [here](https://github.com/aknay/TutorialSyleProject/tree/master/AkkaUdp){:target="_blank"}.

#### **Prerequisites**
Please make sure both Scala and SBT are installed on your local Linux system. Please follow my tutorial post [here]({% post_url 2017-05-09-how-to-install-scala-and-sbt-in-raspberry-pi-3  %}){:target="_blank"} to install. 

#### **Reference**
1. [Akka docs](https://doc.akka.io/docs/akka/2.5/io-udp.html){:target="_blank"}


#### **Overview**
Sometimes we want to use awesome Akka and yet we want to send a message to another system which does not support Akka natively. Then a simple solution is that we send messages using UDP inside Akka. By using UDP, we are free to send datagrams to any destination and still remain connectionless.   

We will create two programs called 
1. ScheduledSenderApp
2. ListenerApp

The `ScheduledSenderApp` will send a message to the `ListenerApp` every one second. Once the `ListenerApp` received the message, it replies back with another message. 

The network setup will be like this,

<div class="mermaid">
graph LR;
   A(ScheduledSenderApp - 127.0.0.1:5115)-->B(ListenerApp - 127.0.0.1:5005)
</div>

> Note: You can also use Python or any other UDP program to test either ScheduledSenderApp or ListenerApp individually.     

#### **Add Dependencies**

We will be using scala version `2.12.4` and `akka-remote` library. Add the following line to the `build.sbt`. You can check the latest version from [here](https://akka.io/docs/){:target="_blank"}.

```
scalaVersion := "2.12.4"
libraryDependencies += "com.typesafe.akka" %% "akka-actor" % "2.5.8"
```

For SBT, we will be using `0.13.16`. Therefore, add the following line to `build.properties`
```
sbt.version=0.13.16
```

> Note: These dependencies are for both Apps.

#### **Scheduled Sender App**
This is the main entry point to run an actor. Here, we assign our local and remote IP address to the `ScheduledSenderActor` Actor class. Next, we will look at this class.


{% highlight java linenos %}
import java.net.InetSocketAddress

import akka.actor._

object ScheduledSenderApp extends App {
  val system = ActorSystem("ActorSystem")
  val remote = new InetSocketAddress("localhost", 5005)
  val local = new InetSocketAddress("localhost", 5115)

  val udp: ActorRef = system.actorOf(ScheduledSenderActor(local, remote), name = "Udp")
}
{% endhighlight %}

#### **Scheduled Sender Actor**
At `line 14`, we bind `self` and `local` address. Once they are bound, `Udp.Bound` at `line 18` will be executed. Then, `context` will switch to `ready` function which is defined at `line  22`. Once all these are done, the scheduler at `line 15` will call `hello` every 1 second. Since `hello` is `String` type, the `hello` command will call `case msg: String` at `line 23` and it will send the message to `remote` using `Udp.Send`

Once we get received data from some remote address, we will print them out as shown in `line  26 to 29`.

At `line 33`, It is a constructor for the `ScheduledSenderActor` class.    

{% highlight java linenos %}
import java.net.InetSocketAddress

import akka.actor.{Actor, ActorLogging, ActorRef, Cancellable, Props}
import akka.io.{IO, Udp}
import akka.util.ByteString
import scala.concurrent.ExecutionContext.Implicits.global

import scala.concurrent.duration._

class ScheduledSenderActor(local: InetSocketAddress, remote: InetSocketAddress) extends Actor with ActorLogging {

  import context.system

  IO(Udp) ! Udp.Bind(self, local)
  val scheduleCancellable: Cancellable = system.scheduler.schedule(0.seconds, 1.second, self, "hello")

  def receive = {
    case Udp.Bound(_) ⇒
      context.become(ready(sender()))
  }

  def ready(send: ActorRef): Receive = {
    case msg: String ⇒
      send ! Udp.Send(ByteString(msg), remote)

    case Udp.Received(data, remoteAddress) ⇒
      val ipAddress = remoteAddress.getAddress.getHostAddress
      val port = remoteAddress.asInstanceOf[InetSocketAddress].getPort
      log.info(s"we received ${data.utf8String} from IP Address: $ipAddress and port number: $port")
  }
}

object ScheduledSenderActor {
  def apply(local: InetSocketAddress, remote: InetSocketAddress) = Props(classOf[ScheduledSenderActor], local, remote)
}
{% endhighlight %}


#### **Listener App**
Similar to the `ScheduledSenderApp` class but we only swap port number.

{% highlight java linenos %}
import java.net.InetSocketAddress

import akka.actor.{ActorRef, ActorSystem}

object ListenerApp extends App  {
  val system = ActorSystem("RemoteActorSystem")
  val local = new InetSocketAddress("localhost", 5005)
  val remote = new InetSocketAddress("localhost", 5115)

  val udp: ActorRef = system.actorOf(ListenerActor(local, remote), name = "Udp")
}
{% endhighlight %}

#### **Remote actor**
It is also similar to the `ScheduledSenderActor` class. We just send the message back to remote at `line 25`.

{% highlight java linenos %}
import java.net.InetSocketAddress

import akka.actor.{Actor, ActorLogging, ActorRef, Props}
import akka.io.{IO, Udp}
import akka.util.ByteString

class ListenerActor(local: InetSocketAddress, remote: InetSocketAddress) extends Actor with ActorLogging {

  import context.system
  IO(Udp) ! Udp.Bind(self, local)

  def receive = {
    case Udp.Bound(_) ⇒
      context.become(ready(sender()))
  }

  def ready(send: ActorRef): Receive = {
    case msg: String ⇒
      send ! Udp.Send(ByteString(msg), remote)

    case Udp.Received(data, remoteAddress) ⇒
      val ipAddress = remoteAddress.getAddress.getHostAddress
      val port = remoteAddress.asInstanceOf[InetSocketAddress].getPort
      log.info(s"we received ${data.utf8String} from IP Address: $ipAddress and port number: $port")
      send ! Udp.Send(ByteString("Hello back"), remote)
  }
}

object ListenerActor {
  def apply(local: InetSocketAddress, remote: InetSocketAddress) = Props(classOf[ListenerActor], local, remote)
}
{% endhighlight %}

> Note: At `line 25`, you can use either use `remote` or `remoteAddress` to reply back. The `remoteAddress` is holding an address where we received data from at that point in time.  

#### **Result**
Once you use click `Run` or `sbt run`, you will receive these messages.

At Scheduled Sender App,
```
... we received Hello back from IP Address: 127.0.0.1 and port number: 5005
```

At Listener App,
```
... we received hello from IP Address: 127.0.0.1 and port number: 5115
```