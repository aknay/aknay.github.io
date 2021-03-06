---
layout: post
title:  "Akka Stream in Bite Size - Part 1 (Basic)"
date:   2017-06-29 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will use some of the basic APIs from Akka Stream as quick start guide. All the following programs are small and runnable. So it is really easy to get started and to get it run. 
<!-- excerpt -->

#### **Prerequisites**
1. Please make sure both Scala and SBT are installed on your system. Please follow my tutorial post [here]({% post_url 2017-05-09-how-to-install-scala-and-sbt-in-raspberry-pi-3  %}) to install. This is applicable for both Raspberry Pi or any Linux system.

#### **Reference**

1. The Akka Steam docs is really good way to start and it can be found [here](http://doc.akka.io/docs/akka/2.5.3/scala/stream/stream-quickstart.html#stream-quickstart){:target="_blank"}.

2. This stackoverflow [post](https://stackoverflow.com/questions/35120082/how-to-get-started-with-akka-streams){:target="_blank"} is really helpful.


#### **Dependency**
I am using scala version `2.12.2`. I added my dependency as `"com.typesafe.akka" %% "akka-stream" % "2.5.3"` in `build.sbt` file.

#### **Source**

A Source is a data generator and initial input to a Stream. It has *exaclty* one `output`. At line `16`, we are creating a source which will generate from 1 to 10. Then we use `runForEach` method to run the `source` at line `17`. Next, we use `terminate` method at line `18` to terminate the system. Normally, the default Akka system will never terminate. 

<div class="mermaid">
graph LR;
   A(Source)-->B(downstream)
   style A fill:#27ae60  ,stroke:#333,stroke-width:4px;
</div>

{% highlight scala linenos %}
package example

import akka.{Done, NotUsed}
import akka.actor.ActorSystem
import akka.stream.ActorMaterializer
import akka.stream.scaladsl.Source

import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val source: Source[Int, NotUsed] = Source(1 to 10)
  val done: Future[Done] = source.runForeach { i => println(i) }
  terminate(done)

  def terminate[T](v: Future[T]): Unit = v.onComplete(_ => system.terminate())
}
{% endhighlight %}
The result is: 

```
1
2
3
4
5
6
7
8
9
10
```

#### **Sink**
A Sink is a data consumer and it locates at the endpoint of a stream. It has *exaclty* one `input`. At line `13`, a sink is created and its job is to print received data from a source. Then we use a chain of commands to create a `flow`.  Once we run that flow, the data from `source` will be flow to `sink`. 
<div class="mermaid">
graph LR;
   A(upstream)-->B(Sink)
   style B fill:#27ae60  ,stroke:#333,stroke-width:4px;
</div>
{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.ActorMaterializer
import akka.stream.scaladsl.{Sink, Source}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val source = Source(1 to 10)
  val sink = Sink.foreach[Int](v => println(s"received: $v"))
  val flow = source to sink
  flow.run
}
{% endhighlight %}

The result is:

```
received: 1
received: 2
received: 3
received: 4
received: 5
received: 6
received: 7
received: 8
received: 9
received: 10
```

#### **Flow**
A Flow serves as a connection between upstream and downstream. It can also transform data elements flowing through it. Here, we use `via` to connect multiple `flows` at line `16`. Noticed that flow `multiplier` is reused at the fourth stage. This is one of the nice things about Akka. It gives you greater reusability power.

<div class="mermaid">
graph LR;
   A(upstream)-->B(Flow)
   B-->C(downstream)
   style B fill:#27ae60  ,stroke:#333,stroke-width:4px;
</div>

{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.ActorMaterializer
import akka.stream.scaladsl.{Flow, Sink, Source}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val source = Source(1 to 10)
  val sink = Sink.foreach[Int](v => println(s"received: $v"))
  val multiplier = Flow[Int].map(v => v * 2)
  val adder = Flow[Int].map(v => v + 1)
  val flow = source via multiplier via adder via multiplier to sink
  flow.run
}
{% endhighlight %}

The result is

```
received: 6
received: 10
received: 14
received: 18
received: 22
received: 26
received: 30
received: 34
received: 38
received: 42
```
#### **Translation of Your Flow Idea with Graphs**
Here, we are using `GrpahDSL` to express the data flow from previous example with the squiggly symbol `~>`.  These symbols help you to visualize the overall structure easily. It is really like the graphical representation of a flow from one end to another. 


<div class="mermaid">
graph LR;
   A(source)-->B(multiplier)
   B-->C(adder)
   C-->D(multiplier)
   D-->E(sink)
</div>

{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape}
import akka.stream.scaladsl.{Flow, GraphDSL, RunnableGraph, Sink, Source}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>
    import akka.stream.scaladsl.GraphDSL.Implicits._

    val source = Source(1 to 10)
    val sink = Sink.foreach[Int](v => println(s"received: $v"))
    val multiplier = Flow[Int].map(v => v * 2)
    val adder = Flow[Int].map(v => v + 1)
    val flow = source ~> multiplier ~> adder ~> multiplier ~> sink
    ClosedShape
  })

  g.run()
}
{% endhighlight %}

The result is

```
received: 6
received: 10
received: 14
received: 18
received: 22
received: 26
received: 30
received: 34
received: 38
received: 42
```

#### **Throttle** 

Before I let you go, we still have one more program to run, my friend. In this program, the data are generated in a timely manner. Let's look at the example. At line `15`, the `source` is generating `5 elements` per `10 second` and burst rate is `2`. You can see from the result that the first two elements are generated in a burst. That's why they are almost in the same second. Once the burst is done, the subsequence element is generated in every two seconds. Why? It is because of the `5 elements per 10 seconds`. For the case of a fast `source` with slow `sink`, we can use this method to back-pressure a up-stream.

{% highlight scala linenos %}
package example

import java.util.Calendar

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ThrottleMode}
import akka.stream.scaladsl.Source
import scala.concurrent.duration._

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val source = Source(1 to 10).throttle(5, 10.second, 2, ThrottleMode.shaping)
  source.runForeach(x => println(x + ": " + Calendar.getInstance().getTime))
}

{% endhighlight %}

The result is:
```
1: Fri Jun 30 16:38:45 JST 2017
2: Fri Jun 30 16:38:45 JST 2017
3: Fri Jun 30 16:38:47 JST 2017
4: Fri Jun 30 16:38:49 JST 2017
5: Fri Jun 30 16:38:51 JST 2017
6: Fri Jun 30 16:38:53 JST 2017
7: Fri Jun 30 16:38:55 JST 2017
8: Fri Jun 30 16:38:57 JST 2017
9: Fri Jun 30 16:38:59 JST 2017
10: Fri Jun 30 16:39:01 JST 2017
```