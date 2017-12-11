---
layout: post
title:  "Akka Remote Messaging With Raspberry Pi"
date:   2017-12-08 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will create an Akka Remote system and send a message between localhost (Ubuntu/Raspberry Pi) and the remote host (Raspberry Pi).
<!-- excerpt -->

#### **Git Hub Project**
The complete project can be cloned from [here](https://github.com/aknay/TutorialSyleProject/tree/master/AkkaRemote){:target="_blank"}.

#### **Prerequisites**
Please make sure both Scala and SBT are installed on your local Linux system and Raspberry Pi. Please follow my tutorial post [here]({% post_url 2017-05-09-how-to-install-scala-and-sbt-in-raspberry-pi-3  %}){:target="_blank"} to install. 

#### **Reference**
1. [Akka docs](https://doc.akka.io/docs/akka/2.5.4/scala/remoting.html){:target="_blank"}
2. [Remote Example](https://alvinalexander.com/scala/simple-akka-actors-remote-example){:target="_blank"}


#### **Add Dependencies**

We will be using scala version `2.12.4` and `akka-remote` library. Add the following line to the `build.sbt`. You can check the latest version from [here](https://akka.io/docs/){:target="_blank"}.

```
scalaVersion := "2.12.4"
libraryDependencies += "com.typesafe.akka" %% "akka-remote" % "2.5.8"
```

For SBT, we will be using `0.13.16`. Therefore, add the following line to `build.properties`
```
sbt.version=0.13.16
```

> Note: These dependencies are for both local and remote host.

#### **Overall Project layout**
This is the overall layout for our project (both local and remote host). If you are starting from the scratch, you will need to follow this layout. Otherwise, you could just clone the entire project and make a modification to it.

![Loal Actor IntelliJ Dir]({{ site.url }}/assets/2017-12-08-IntelliJ-dir.png)

#### **Overall Setup**
Before we begin, I'll just brief what we are going to do. First, we will create a `configuration file` for both local (Ubuntu) and remote (Raspberry Pi). The local system uses `127.0.0.1` and the remote system uses `192.168.1.101`. Both local and remote systems are on the same network connected via a router. The local machine sends a message every 1 second and the remote machine replies back when it receives from the local machine.  

<div class="mermaid">
graph LR;
   A(local host - 127.0.0.1)-->B(remote host - 192.168.1.101)
</div>

#### **Configuration File**
To enable remote messaging in our project, we should create and add an `application.conf` file under `<project folder>/src/main/resources` folder. You can look up our project layout again that I mentioned above. 

#### **Local Actor Configuration File**
Remember that local machine will use `127.0.0.1`. We don't really particular about port number here. Therefore, we just use `0` and the system will use any available port. 

{% highlight java linenos %}
akka {
  actor {
    provider = remote
  }
  local {
    enabled-transports = ["akka.remote.netty.tcp"]
    netty.tcp {
      hostname = "127.0.0.1"
      port = 0
    }
  }
}
{% endhighlight %}

#### **Local actor**

At `line 7`, we use `context.actorSelection` to look for the remote machine IP address and Actor name. At `line 11`, we are using `scheduler` to send a message every one second to the remote machine. 

{% highlight java linenos %}
import akka.actor.{Actor, ActorSelection, Cancellable}
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.duration._

class LocalActor extends Actor {

  val remote: ActorSelection = context.actorSelection("akka.tcp://RemoteActorSystem@192.168.1.101:1212/user/RemoteActor")

  val TalkToRemote = "talkToRemote"

  val scheduleCancellable: Cancellable = context.system.scheduler.schedule(0.seconds, 1.second, self, TalkToRemote)

  def receive = {
    case TalkToRemote =>
      remote ! "Hello from the Local Actor"

    case msg: String =>
      println(s"LocalActor received message: '$msg'")
  }
}
{% endhighlight %}

#### **Local actor App**
This is the main entry point to run our `Local Actor` application. At `line 5`, we just use `LocatActor` class to instantiate as an actor. 

{% highlight java linenos %}
import akka.actor.{ActorSystem, Props}

object LocalActorApp extends App {
  implicit val system = ActorSystem("LocalSystem")
  val localActor = system.actorOf(Props[LocalActor], name = "LocalActor")
}
{% endhighlight %}


#### **Remote Actor Configuration File**
Please remember that we set up our remote host (Raspberry Pi) IP address as `192.168.1.101`. You should change to the different IP address in this configuration file according to your Raspberry Pi IP address. The port number should be same for the port number that we defined in `ActorSelection` at `Local Actor`.

{% highlight java linenos %}
akka {
  actor {
    provider = remote
  }
  remote {
    enabled-transports = ["akka.remote.netty.tcp"]
    netty.tcp {
      hostname = "192.168.1.101"
      port = 1212
    }
  }
}
{% endhighlight %}

#### **Remote actor**
This remote actor is just sending back a message when it receives from `Local Actor`.

{% highlight java linenos %}
import akka.actor.Actor

class RemoteActor extends Actor {
  def receive = {
    case msg: String =>
      println(s"Remote Actor received message '$msg'")
      sender ! "Hello from the Remote Actor"
  }
}
{% endhighlight %}


#### **Remote Actor App**
This is also the main entry point to run our `Remote Actor` application.

{% highlight java linenos %}
import akka.actor._

object RemoteActorApp extends App  {
  val system = ActorSystem("RemoteActorSystem")
  val remoteActor = system.actorOf(Props[RemoteActor], name = "RemoteActor")
}
{% endhighlight %}

#### **Run Remote Actor**
At a terminal, go to the folder (Remote Actor in this case) at Raspberry Pi. Type in `sbt` and then follow by `run` command. Now the remote actor is waiting for messages from the local actor. 

#### **Run Local Actor**
Similarly, go to the folder (Local Actor in this case) at Ubuntu/Linux system. Type in `sbt` and then follow by `run` command. Now local actor started sending at the 1-second interval. 

#### **Result**
At local host terminal, you should see this message at every 1 second.
```
LocalActor received message: 'Hello from the Remote Actor'
```

At remote host (Raspberry Pi) terminal, you should see this message at every 1 second.
```
Remote Actor received message 'Hello from the Local Actor'
```

> Note: If you could not manage to receive any message, you should check IP address of the remote host, configuration file and actor selection. 