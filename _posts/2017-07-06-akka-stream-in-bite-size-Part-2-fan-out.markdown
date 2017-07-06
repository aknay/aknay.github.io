---
layout: post
title:  "Akka Stream in Bite Size - Part 2 (Fan-out)"
date:   2017-07-06 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
In this second part of Akka stream tutorial, we will mainly touch on the how to split a single stream into multiple streams (Fan-out). 
<!-- excerpt -->

#### **Prerequisites**
1. Please follow my first Part 1 tutorial post [here]({% post_url 2017-06-29-akka-stream-in-bite-size-Part-1-basic  %}).

#### **Reference**

1. The Akka Steam [Working With Graphs](http://doc.akka.io/docs/akka/2.5.3/scala/stream/stream-graphs.html){:target="_blank"} docs is really good way to start.


#### **Constructing Graphs with Broadcast (1 input, N outputs)**
This example illustrates how we can split a single source stream into two streams. At line `14`, we create a source using `shoppingList`. This `source` will flow into the broadcast at line `22` and the output will be `sink` to `printItemSink` and `printCostSink` sinks at line `23` and `24` respectively. Those `sinks` are just to print out the element from the broadcast. So the overall process can be realized as the `source` flow into the `broadcast` and then split into two streams based on the element location `_._1 or _._2`. And finally, these two streams go into two `sinks` to generate the result. 

{% mermaid %}
graph LR;
   A(Source)
    A --> B{Broadcast}
    B --> C[Sink]
    B --> D[Sink]
{% endmermaid %}

{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape}
import akka.stream.scaladsl.{Broadcast, Flow, GraphDSL, RunnableGraph, Sink, Source}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val shoppingList: List[(String, Int)] = List(("Keyboard", 5), ("Mouse", 2), ("Monitor", 79), ("Memory", 32))

  val source = Source(shoppingList)
  val printItemSink = Sink.foreach[String](v => println("item: " + v))
  val printCostSink = Sink.foreach[Int](v => println("cost: " + v))

  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>
    import GraphDSL.Implicits._

    val bcast = b.add(Broadcast[(String, Int)](2))
    source ~> bcast.in
    bcast.out(0) ~> Flow[(String, Int)].map(_._1) ~> printItemSink
    bcast.out(1) ~> Flow[(String, Int)].map(_._2) ~> printCostSink
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

#### **Constructing Graphs With Balance (1 input, N outputs)**

In this example, we can not only split a single source into two streams but also it can be balanced between two streams. The `flowWithBackPressure` is added at `line 16`in this example to illustrate the effect of the balancing act. As you can see from the result, most of the items are flowing into the `sinkB` because the `sinkA` is flowing with the backpressure.

{% mermaid %}
graph LR;
   A(Source)
    A --> B{Balance}
    B --> C[Sink]
    B --> D[Sink]
{% endmermaid %}


{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape, ThrottleMode}
import akka.stream.scaladsl.{Balance, Flow, GraphDSL, RunnableGraph, Sink, Source}
import scala.concurrent.duration._

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val source = Source(1 to 10)
  val sinkA = Sink.foreach[Int](x => println(s"Sink A::$x"))
  val sinkB = Sink.foreach[Int](x => println(s"Sink B::$x"))
  val flowWithBackPressure = Flow[Int].throttle(1, 1.second, 1, ThrottleMode.shaping).map(v => v * 2)
  val flowWithoutBackPressure = Flow[Int].map(v => v * 2)

  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>
    import GraphDSL.Implicits._

    val balancer = b.add(Balance[Int](2))

    source ~> balancer
    balancer ~> flowWithBackPressure ~> sinkA
    balancer ~> flowWithoutBackPressure ~> sinkB

    ClosedShape
  })

  g.run()
}
{% endhighlight %}

The result is 
```
Sink A::2
Sink B::4
Sink B::8
Sink B::10
Sink B::12
Sink B::14
Sink B::16
Sink B::18
Sink B::20
Sink A::6
```
#### **Constructing Graphs with UnzipWith (1 input, N outputs)**

This is similar to the Graphs with `Broadcast` but the signature to define the flow is a little bit different. You can treat the `UnzipWith` function at `line 24` as a `split` function. The function can be used for splitting each element from the input stream and sent those elements into multiple downstreams. As compared to the `Broadcast`, the function `UnzipWith[(String, Int),String, Int]` shows exactly how you want to split the elements and how many downstreams. So in this case, we are going to split `(String, Int)` into `String` and `Int` streams.

{% mermaid %}
graph LR;
   A(Source)
    A --> B{UnzipWith}
    B --> C[Sink]
    B --> D[Sink]
{% endmermaid %}



{% highlight scala linenos %}
package example

package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape}
import akka.stream.scaladsl.{GraphDSL, RunnableGraph, Sink, Source, UnzipWith}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val shoppingList: List[(String, Int)] = List(("Keyboard", 5), ("Mouse", 2), ("Monitor", 79), ("Memory", 32))

  val source = Source(shoppingList)
  val printItemSink = Sink.foreach[String](v => println("item: " + v))
  val printCostSink = Sink.foreach[Int](v => println("cost: " + v))

  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>

    import GraphDSL.Implicits._

    val split = b.add(UnzipWith[(String, Int), String, Int]((v: (String, Int)) => (v._1, v._2)))
    
    source ~> split.in
    split.out0 ~> printItemSink
    split.out1 ~> printCostSink

    ClosedShape
  })
  g.run()
}
{% endhighlight %}

The result is
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
#### **Constructing Graphs with Unzip (1 input, 2 outputs)**
This example is similar to the `UnzipWith` but `Unzip` can only split into two streams. Therefore, the declearing of `unzip` function at `line 22` is very simple.


{% mermaid %}
graph LR;
   A(Source)
    A --> B{Unzip}
    B --> C[Sink]
    B --> D[Sink]
{% endmermaid %}

{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape}
import akka.stream.scaladsl.{GraphDSL, RunnableGraph, Sink, Source, Unzip, UnzipWith}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val shoppingList: List[(String, Int)] = List(("Keyboard", 5), ("Mouse", 2), ("Monitor", 79), ("Memory", 32))

  val source = Source(shoppingList)
  val printItemSink = Sink.foreach[String](v => println("item: " + v))
  val printCostSink = Sink.foreach[Int](v => println("cost: " + v))

  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>

    import GraphDSL.Implicits._

    val unzip = b.add(Unzip[String, Int])

    source ~> unzip.in
    unzip.out0 ~> printItemSink
    unzip.out1 ~> printCostSink

    ClosedShape
  })
  g.run()
}
{% endhighlight %}

The result is
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