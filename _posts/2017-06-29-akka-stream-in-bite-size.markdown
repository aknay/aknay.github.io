---
layout: post
title:  "Akka Stream in Bite Size"
date:   2017-06-29 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will use some of the Akka Stream API to understand. All the following program are small and runnable so that it is easier to get started. 
<!-- excerpt -->

### Prerequisites
1. Please make sure both Scala and SBT are installed on your system. Please follow my tutorial post [here]({% post_url 2017-05-09-how-to-install-scala-and-sbt-in-raspberry-pi-3  %}) to install. This is applicable for both Raspberry Pi or any Linux system.

### Reference

1. The Akka Steam docs are really good way to start and it can be found [here](http://doc.akka.io/docs/akka/2.5.3/scala/stream/stream-quickstart.html#stream-quickstart){:target="_blank"}.

2. This stackoverflow [post](https://stackoverflow.com/questions/35120082/how-to-get-started-with-akka-streams){:target="_blank"} is really helpful.


### Dependency
I am using scala version `2.12.2`. I added my dependency as `"com.typesafe.akka" %% "akka-stream" % "2.5.3"` in `build.sbt` file.

### Source

A Source is a data generator and initial input to a Stream. At line `16`, we are creating a source which will generate from 1 to 10. Then we use `runForEach` method to run the `source` at line `17`. Next, we use `terminate` method at line `18` to terminate the system. Normally, the default Akka system will never terminate. 
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

### Sink
A Sink is a data consumer and it locates at the endpoint of a stream. At line `13`, a sink is created and its job is to print received data from a source. Then we use a chain of commands to create a `flow`.  Once we run that flow, the data from `source` will be flow to `sink`. 

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

### Flow
 Here, we use `via` to connect multiple `flows` at line `16`. Noticed that flow `multiplier` is reused at the fourth stage. This is one of the nice things about Akka. It gives you greater reusability power.


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
### Translation of Your Flow Idea with Graphs

The exact same result from the previous example can be reproduced with this example. Better yet, the squiggly symbol `~>` helps you to visualize the overall structure easily. It is really like the graphical representation of a flow from one end to another. 

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

### Broadcast
This example illustrates how we can split a single source stream into two streams. At line `15`, we create a source using two different lists, `zip` it as one single list and pass it to the source. This `source` will flow into the broadcast at line `23` and the output will be `sink` to `printItem` and `printCost` sinks at line `24` and `25`. Those `sinks` are just to print out the element from the broadcast. So the overall process can be realized as the `source` flow into the `broadcast` and then split into two streams based on the element location `_._1 or _._2`. And finally, these two streams go into two `sinks` to generate the result. If you are still not sure, just follow those squiggly symbols. 

{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape}
import akka.stream.scaladsl.{Broadcast, Flow, GraphDSL, RunnableGraph, Sink, Source}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val items = List("Keyboard", "Mouse", "Monitor", "Memory")
  val cost = List(5, 2, 79, 32)
  val shopping: List[(String, Int)] = items zip cost
  val source = Source(shopping)

  val printItem = Sink.foreach[String](v => println("item: " + v))
  val printCost = Sink.foreach[Int](v => println("cost: " + v))
  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>
    import akka.stream.scaladsl.GraphDSL.Implicits._

    val bcast = b.add(Broadcast[(String, Int)](2))
    source ~> bcast.in
    bcast.out(0) ~> Flow[(String, Int)].map(_._1) ~> printItem
    bcast.out(1) ~> Flow[(String, Int)].map(_._2) ~> printCost
    ClosedShape
  })
  g.run()
}
{% endhighlight %}

The result is:

```
item: Keyboard
cost: 5
item: Mouse
cost: 2
item: Monitor
cost: 79
item: Memory
cost: 32
```

### Throttle 

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