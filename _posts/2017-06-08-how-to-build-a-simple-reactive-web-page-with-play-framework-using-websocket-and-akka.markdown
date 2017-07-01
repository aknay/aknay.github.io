---
layout: post
title:  "How to build a simple reactive web page with Play framework using WebSocket and Akka"
date:   2017-06-08 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will use Play framework to build a simple reactive web page using WebSocket and Akka. The completed git repo is provided. Let's get digging.

<!-- excerpt -->

#### **The Complete Project**
You can look at the working example from my [Github](https://github.com/aknay/scala-play-websocket-akka){:target="_blank"} repo. 

#### **Prerequisites**
1. Please make sure both Scala and SBT are installed on your system. Please follow my tutorial post [here]({% post_url 2017-05-09-how-to-install-scala-and-sbt-in-raspberry-pi-3  %}) to install. This is applicable for both Raspberry Pi or any Linux system.

2. You also need to know a little about how WebSocket works. You can play with WebSocket from [here](http://www.websocket.org/echo.html){:target="_blank"}.

3. Finally, it is good to know how to send a message in Akka.  

#### **Reference**

1. The docs of Scala WebSocket can be found [here](https://www.playframework.com/documentation/2.5.x/ScalaWebSockets){:target="_blank"}.
2. You can find how to send a message using Akka from [here](http://doc.akka.io/docs/akka/2.5.2/scala/actors.html#send-messages){:target="_blank"}.

#### **Home Controller**

First, Let's look at the `HomeController.scala`, There are two functions `index` and `ws`. The `index` function will host the `index.scala.html` and `ws` function will take care of WebSocket connection. Once WebSocket is established, Play framework will pass `ActorRef` to `ws` function. Then we will pass this `ActorRef` to our own `WebSocketActor` to generate `Props` to pass back to `ActorFlow`. 


##### HomeController.scala

{% highlight scala linenos %}
@Singleton
class HomeController @Inject()(implicit actorSystem: ActorSystem,
                               mat: Materializer
                              ) extends Controller {

  // Home page that renders template
  def index = Action { implicit request =>
    Ok(views.html.index())
  }

  def ws: WebSocket = WebSocket.accept[String, String] { request =>
    ActorFlow.actorRef(out => WebSocketActor.props(out))
  }
{% endhighlight %}

#### **WebSocketActor**
Once we get the `ActorRef` from Play framework, we can send WebSocket message back. Now let's look at `WebSocketActor.scala`. Using the Akka, we can simple send a message back to WebSocket using `out ! "your websocket message"`. Becuase I want to send internally and periodically, the `scheduler` is added in this Scala program. So every 1000 millis, the scheduler will send `SendLatestMessage` case object to `WebSocketActor`. This `SendLatestMessage` will trigger to send WebSocket message using `out` as `ActorRef`. You can refer to this line `out ! "Displaying message from Akka at " + Calendar.getInstance().getTime.toString`.
##### WebScoketActor.scala
{% highlight scala linenos %}
object WebSocketActor {
  def props(out: ActorRef): Props = Props(new WebSocketActor(out))
}

class WebSocketActor(out: ActorRef) extends Actor with ActorLogging {
  val tick: Cancellable = {
    context.system.scheduler.schedule(Duration.Zero, 1000.millis, self, SendLatestMessage)(context.system.dispatcher)
  }

  def receive = {
    case SendLatestMessage =>
      log.info("displaying")
      out ! "Displaying message from Akka at " + Calendar.getInstance().getTime.toString
    case msg: String =>
      out ! "I renceived your message: " + msg + " at " + Calendar.getInstance().getTime.toString
  }

}

case object SendLatestMessage
{% endhighlight %}
#### **HTML and CoffeeScript**

Now, let's look at how to establish WebSocket at both `HTML` and `CoffeeScript` files. As you may notice in the HTML file, `@routes.HomeController.ws.webSocketURL()` is URL link for WebSocket. It is same as `ws://localhost:9000/ws` if your are running locally. The CoffeeScript will get that URL and create a new WebSocket. You can refer to this line `ws = new WebSocket $("body").data("ws-url")`. Once data is received from Akka, the `ws.onmessage` will be called and display a message on `message_holder` HTML tag. 

##### index.scala.html
{% highlight html linenos %}
@()(implicit r: Request[_])
<!DOCTYPE html>
<html>
<head>
    <title>Reactive Website</title>
    <script type='text/javascript' src='@routes.Assets.at("lib/jquery/jquery.min.js")'></script>
    <script type='text/javascript' src='@routes.Assets.at("javascripts/index.js")'></script>
</head>
<body data-ws-url="@routes.HomeController.ws.webSocketURL()">
    <div id="message_holder"></div>
</body>
</html>
{% endhighlight %}

##### index.coffee 
{% highlight javascript linenos %}
$ ->
  ws = new WebSocket $("body").data("ws-url")

  ws.onmessage = (event) ->
    message = event.data
    display(message)
    console.log(message)

  ws.onopen = (event) ->
    console.log("opened")

  ws.onclose = (event) ->
    console.log("close")

  ws.onerror = (event) ->
    console.log("error")

  display = (message) ->
    $("#message_holder").text(message)
{% endhighlight %}

#### **Testing**

Again, you can just download the completed project from [here](https://github.com/aknay/scala-play-websocket-akka){:target="_blank"}. Once it is downloaded, you can simply run using `$ sbt run`. Go to any browser and access `http://localhost:9000/`. You will see a message every one second.