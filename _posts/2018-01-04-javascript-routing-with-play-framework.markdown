---
layout: post
title:  "JavaScript Routing with Play Framework"
date:   2018-1-04 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this tutorial, we will create a Play Framework (Scala) application using JavaScript Routing (Ajax + jQuery) to communicate with a backend server without reloading the page .
<!-- excerpt -->

#### **Git Hub Project**
The complete project can be cloned from [here](https://github.com/aknay/TutorialSyleProject/tree/master/AkkaRemote){:target="_blank"}.

#### **Prerequisites**
Please make sure both Scala and SBT are installed on your local Linux system. Please follow my tutorial post [here]({% post_url 2017-05-09-how-to-install-scala-and-sbt-in-raspberry-pi-3  %}){:target="_blank"} to install. 

#### **Reference**
1. [Play Framework docs about JavaScript Routing (Scala)](https://www.playframework.com/documentation/2.6.x/ScalaJavascriptRouting){:target="_blank"}
2. [Greate Explanation from StackOverFlow](https://stackoverflow.com/questions/11133059/play-2-x-how-to-make-an-ajax-request-with-a-common-button){:target="_blank"}


#### **Overall Setup**
You can download and start from the scratch using the starter example from [here](https://github.com/playframework/play-scala-starter-example) or you can just download the completed project from [here]() and play with it.

Briefly, 
1. We need controller class to define our controller actions
2. We define these actions in `JavaScriptReverseRouter` 
3. We will also need to define these actions in `routes` (just like any routes in Play framework)
4. We will use JavaScript functions to call these routes from a web page
5. After successful call, we will display on the web page
6. In between them, we will also use dummy storage object to store data


#### **Add Dependencies**

We will be using scala version `2.12.4`. We will add `guice` and `play test` just like any play framework. Since we will be calling `Ajax` request using `JavaScript`, we will just add `jquery` to our dependencies to make our life easier.  

```
scalaVersion := "2.12.4"

libraryDependencies += guice
libraryDependencies += "org.scalatestplus.play" %% "scalatestplus-play" % "3.1.2" % Test
libraryDependencies += "org.webjars" % "jquery" % "2.1.3"
```


#### **Home Controller**

We define `getName` and `updateName` actions in `HomeController` class. At `getName` action, we get the name from the storage, add to the Json and then make the response with this Json. At `updateName` action, we get the name from our user and save it to our storage. Then we just again response with Json. In order to call these routes from JavaScript, we need to generate JavaScript Routing. These can be simply done by just adding in `jsRoutes` action.


{% highlight java linenos %}
package controllers

import javax.inject._

import play.api.libs.json.Json
import play.api.mvc._
import play.api.routing.JavaScriptReverseRouter
import services.NameStorage

import scala.concurrent.Future

@Singleton
class HomeController @Inject()(cc: ControllerComponents) extends AbstractController(cc) {

  def index = Action {
    Ok(views.html.index("Ajax Play Application"))
  }

  def getName = Action.async { implicit request =>
    Future.successful(Ok(Json.toJson(NameStorage.getName)))
  }

  def updateName(name: String) = Action.async { implicit request =>
    NameStorage.setName(name)
    Future.successful(Ok(Json.toJson(NameStorage.getName)))
  }

  def jsRoutes = Action { implicit request =>
    Ok(
      JavaScriptReverseRouter("jsRoutes")(
        routes.javascript.HomeController.getName,
        routes.javascript.HomeController.updateName
      )).as("text/javascript")
  }

}
{% endhighlight %}

### **Router**
We will add all these actions to our router file `routes`

{% highlight java linenos %}
GET     /                             controllers.HomeController.index
GET     /person                       controllers.HomeController.getName
GET     /person/:name                 controllers.HomeController.updateName(name: String)
GET     /jsr                          controllers.HomeController.jsRoutes

# Map static resources from the /public folder to the /assets URL path
GET     /assets/*file                 controllers.Assets.versioned(path="/public", file: Asset)
{% endhighlight %}


#### **Index Html**

The important part of this HTML is that make sure these are added
1. jQuery library (`jquery.min.js`)
2. JavaScript file that we gonna use (in this case: `hello.js`)
3. JavaScript Route (here: `@routes.HomeController.jsRoutes`)
4. Matching Html Id with `hello.js` file

{% highlight java linenos %}
@(title: String)

<!DOCTYPE html>
<html lang="en">
<head>
 <title>@title</title>
 <link rel="stylesheet" media="screen" href="@routes.Assets.versioned("stylesheets/main.css")">
 <link rel="shortcut icon" type="image/png" href="@routes.Assets.versioned("images/favicon.png")">
 <script type="text/javascript" src="@routes.Assets.versioned("lib/jquery/jquery.min.js")"></script>
        <script src="@routes.Assets.versioned("javascripts/hello.js")" type="text/javascript"></script>
 <script type="text/javascript" src="@routes.HomeController.jsRoutes"></script>
</head>

<body>
<span> Current Name is: </span>
<span  id="name"></span>
<br>
<input type="button" value="Set Name" id="set_name" >
<input type="text" value="Jane" id="input_name">
<br>
<input type="button" value="Get Name" id="get_name" >

</body>
</html> 
{% endhighlight %}

#### **JavaScript** 

You should take note how the ajax requests are called here.
1. `jsRoutes.controllers.HomeController.getName().ajax(/*...*/)`
2. `jsRoutes.controllers.HomeController.updateName(name).ajax(/*...*/)`


{% highlight java linenos %}
$(document).ready(function() {
    $('#get_name').click(function() {
        jsRoutes.controllers.HomeController.getName().ajax({
            success: function(result) {
                alert("Hello: " + result)
                $("#name").text(result);
            },
            failure: function(err) {
                var errorText = 'There was an error';
                $("#name").text(errorText);
            }
        });
    });

    $('#set_name').click(function() {
        var inputName = $("#input_name").val()
        jsRoutes.controllers.HomeController.updateName(inputName).ajax({
            success: function(result) {
                $("#name").text(result);
            },
            failure: function(err) {
                var errorText = 'There was an error';
                $("#name").text(errorText);
            }
        });
    });

});
{% endhighlight %}

#### **Storage**

This is just a dummy database to store a name. 

{% highlight java linenos %}

package services

object NameStorage {
  private var mName = "Bob"

  def setName(name: String): Unit = {
    mName = name
  }

  def getName: String = mName
}
{% endhighlight %}

#### **Result**
1. Go to project folder and do `sbt run`
2. Go to `localhost:9000` using any browser.
3. Click `GetName` Button. There will be an alert message with `Bob` name if it is the first time loading. You can also change the name at input box. Then clicking on `Set Name` should change the current name automatically using refreshing the page.