---
layout: post
title:  "Akka Stream in Bite Size - Part 3 (Fan-In)"
date:   2017-07-12 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
In this third part of Akka stream tutorial, we will focus on the how to combine multiple streams into a single stream(Fan-In). 
<!-- excerpt -->

#### **Prerequisites**
1. Please follow my first Part 1 tutorial post [here]({% post_url 2017-06-29-akka-stream-in-bite-size-Part-1-basic  %}).

#### **Reference**

1. The Akka Steam [Working With Graphs](http://doc.akka.io/docs/akka/2.5.3/scala/stream/stream-graphs.html){:target="_blank"} docs is really good way to start.


#### **Constructing Graphs with Merge (N inputs, 1 output)**
This example illustrates how we can merge multiple streams into a single stream. The `merge` function at `line 20` will picks elements `randomly` from input sources and push those elements `one by one` into downstream.

<div class="mermaid">
graph LR;
   A(Source) --> C{Merge}
   B(Source) --> C{Merge}
   C --> D[Sink]
</div>

{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape}
import akka.stream.scaladsl.{GraphDSL, Merge, RunnableGraph, Sink, Source}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val sink = Sink.foreach(println)
  val source1 = Source(1 to 4)
  val source2 = Source(5 to 8)
  
  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>

    import GraphDSL.Implicits._

    val merge = b.add(Merge[Int](2))

    source1 ~> merge ~> sink
    source2 ~> merge

    ClosedShape
  })
  g.run()
}
{% endhighlight %}

The result is:

```
1
5
2
6
3
7
4
8
```

#### **Constructing Graphs With MergePreferred (N inputs + Perferred input , 1 output)**

The `MergePreferred` function is similar to `Merge` but the `MergePerferred` has one additional port as `perferred` port. The `MergerPerferred` function picks elements from `perferred` port first. If there is no element availabe from perferred port, then the `MergePreferred` function starts to pick elements from `other` ports.

At `line 21`, we declear `MergePreferred` to merge two input sources. At `line 23`, we mark the `source1` as perferred source. As you can see from the result, the initial values are printed from all three sources. After that, values from `source1` flow into the `sink` since we declear the `source1` as `perferred` port. Once elements are depleted in `source1`, the elements from `source2` and `source3` are flow into the `sink`. 

<div class="mermaid">
graph LR;
   A(Perferred Source) --> D{MergePreferred}
   B(Source) --> D{MergePreferred}
   C(Source) --> D{MergePreferred}
   D --> E[Sink]
</div>


{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape}
import akka.stream.scaladsl.{GraphDSL, MergePreferred, RunnableGraph, Sink, Source}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val source1 = Source(10 to 15)
  val source2 = Source(20 to 25)
  val source3 = Source(30 to 35)
  val sink = Sink.foreach(println)

  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>

    import GraphDSL.Implicits._

    val merge = b.add(MergePreferred[Int](2))

    source1 ~> merge.preferred

    source2 ~> merge
    source3 ~> merge
    merge.out ~> sink

    ClosedShape
  })
  g.run()
{% endhighlight %}

The result is 
```
10
20
30
11
12
13
14
15
21
31
22
32
23
33
24
34
25
35
```
#### **Constructing Graphs with zipWith (N outputs, 1 input)**
This is just a reverse of `UnzipWith` in `Fan-out` Example in Part 2. The core idea lies at `line 20`. The `line 20` can be interpreted as 'The `String` and  `Int` sources are zipped as `(String, Int)` tuple by using the function `(i, s) => (i, s)`'. 

<div class="mermaid">
graph LR;
   A(Source) --> C{zipWith}
   B(Source) --> C{zipWith}
   C --> D[Sink]
</div>



{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape}
import akka.stream.scaladsl.{GraphDSL, RunnableGraph, Sink, Source, ZipWith}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val itemsSource = Source(List("Keyboard", "Mouse", "Monitor", "Memory"))
  val costSource = Source(List(5, 2, 79, 32))
  val sink = Sink.foreach[(String, Int)](v => println("Pair item is: " + v))

  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>

    import GraphDSL.Implicits._

    val zipWith = b.add(ZipWith[String, Int, (String, Int)]((i, s) => (i, s)))

    itemsSource ~> zipWith.in0
    costSource ~> zipWith.in1

    zipWith.out ~> sink

    ClosedShape
  })
  g.run()
}
{% endhighlight %}

The result is
```
Pair item is: (Keyboard,5)
Pair item is: (Mouse,2)
Pair item is: (Monitor,79)
Pair item is: (Memory,32)
```
#### **Constructing Graphs with zip (2 inputs, 1 output)**
This example is similar to the `zipWith` but `zip` can only accept 2 inputs. Therefore, the declearing of `zip` function at `line 20` is very simple as compared to `zipWith` function. This example result is exactly same as the `zipWith` example.


<div class="mermaid">
graph LR;
   A(Source)
   A(Source) --> C{Zip}
   B(Source) --> C{Zip}
   C --> D[Sink]
</div>

{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape}
import akka.stream.scaladsl.{GraphDSL, RunnableGraph, Sink, Source, Zip}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val itemSource = Source(List("Keyboard", "Mouse", "Monitor", "Memory"))
  val costSource = Source(List(5, 2, 79, 32))
  val sink = Sink.foreach[(String, Int)](v => println("Pair item is: " + v))

  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>

    import GraphDSL.Implicits._

    val zip = b.add(Zip[String, Int]())
    costSource ~> zip.in1
    itemSource ~> zip.in0
    zip.out ~> sink
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
#### **Constructing Graphs with Concat (2 inputs, 1 output)**
This function quite simple to work with. The `Concat` function concatenates two streams by consuming first stream,  then the second stream. As you can see from the result, the elments from the `source1` flow into the `sink` first. Once it is done, the elements from the `source2` flow into the `sink`.


<div class="mermaid">
graph LR;
   A(Source)
   A(Source) --> C{Concat}
   B(Source) --> C{Concat}
   C --> D[Sink]
</div>

{% highlight scala linenos %}
package example

import akka.actor.ActorSystem
import akka.stream.{ActorMaterializer, ClosedShape}
import akka.stream.scaladsl.{Concat, GraphDSL, RunnableGraph, Sink, Source}

object AkkaStreamExamples extends App {

  implicit val system = ActorSystem("system")
  implicit val materializer = ActorMaterializer()

  val source1 = Source(1 to 5)
  val source2 = Source(10 to 15)
  val sink = Sink.foreach(println)

  val g = RunnableGraph.fromGraph(GraphDSL.create() { implicit b =>

    import GraphDSL.Implicits._

    val concat = b.add(new Concat[Int](2))
    source1 ~> concat.in(0)
    source2 ~> concat.in(1)
    concat.out ~> sink

    ClosedShape
  })

  g.run()
}
{% endhighlight %}

The result is
```
1
2
3
4
5
10
11
12
13
14
15
```