---
layout: post
title:  "Kotlin in Bite Size - Function"
date:   2017-08-11 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this Kotlin tutorial series, we will expore how to write a concise and expressive function using Kotlin programming language.
<!-- excerpt -->

#### **Prerequisites**
1. Please make sure Java and IntelliJ IDE are installed on your system. 

#### **Reference**
1. `Kotlin in Action` Book by Dmitry Jemerov and Svetlana Isakova
2.  [The official Kotlin reference](https://kotlinlang.org/docs/reference/functions.html){:target="_blank"} 

#### **Hello World**
Let's just create a Kotlin project using IntelliJ IDE. Goto `File > New > Project > Kotlin > Kotlin(JVM)`. Type in your project name and click `Finish`. Inside the project tree, `right click` on `src` and choose `New > Kotlin File/Class`. Key in your preferred file name and click `Ok`. Then paste the following code inside that newly created file. There will be Kotlin icon appear beside `fun main(args: Array<String>)`. Click on it and the IDE will setup everything for you.
{% highlight kotlin linenos %}
fun main(args: Array<String>) {
    println("Hello, world!")
}
{% endhighlight %}

Once you run the code, you will see `Hello, world!` on IDE command window.

From the above syntax, you can observe that

1. The `fun` is keyword for declaring a function
2. Function parameters are defined using `Pascal` notation. Meaning: `name: type`
3. The `println` is a short form of `System.out.println` in Java.
4. There is no `;` at the end of the statement. 


#### **Basic Function**
Let's just create a function. Then we call that function from the `main` 

{% highlight kotlin linenos %}
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}

fun main(args: Array<String>) {
    println(max(2,5))
}
{% endhighlight %}

Once you run that function, the system will print out as `5`. Quite simple, right?
From the above syntax, the return type is defined as `Int`. It is defined by using `: Int` at the end of this line `fun max(a: Int, b: Int): Int {`  

Let's modify the same function to become more expressive.

{% highlight kotlin linenos %}
fun max(a: Int, b: Int): Int = if (a > b) a else b
{% endhighlight %}

Now you will notice that there is no `return` keyword and `{...}` is replaced with `=`. Now the function becomes `expression body`

Let's modify further by removing the `return type`. Here we don't define the return type explicitly and let the compiler figure out by itself. By doing so, we let the compiler to do `type inference` for the return type. 

{% highlight kotlin linenos %}
fun max(a: Int, b: Int) = if (a > b) a else b
{% endhighlight %}

#### **Function using Infix Notation**

We will use `data class` for vector addition in this example. Data classes make the code more readable by using a meaningful name for properties. Notice that there is an `infix` notation in front of `fun` in line 3.

{% highlight kotlin linenos %}
data class Vector (val x: Int, val y: Int)

infix fun Vector.plus(v: Vector) = Vector(this.x + v.x, this.y + v.y)

fun main(args: Array<String>) {
    val v1 = Vector(5,10)
    val v2 = Vector(2, 3)
    val resultant = v1 plus v2
    println(resultant)
}
{% endhighlight %}

The IDE will print out as `Vector(x=7, y=13)` after we run this program. Here, `plus` function is acting like overriding operator function. Therefore, we can easily add two vectors by using `plus` function. There are some constraints when calling a function using infix notation. One of them is that the function should only has a single parameter.

#### **Default and Named Argument**

Function parameters can have default value and parameters can be named when calling a function. Let's look at the following example. Notice the `$` in line 2. Here, we are using `string templates` to print out day, month and year. So when we are calling the function, the default 2017 will be used if we don't specify the `year` parameters. 

When we are calling function, we can just use the parameter name and assign directly without following the parameter sequence as shown in line 6. 

{% highlight scala linenos %}

fun today(day: Int, month: String, year: Int = 2017) {
    println("Today is $day/$month/$year")
}

fun main(args: Array<String>) {
    today(month = "July", day = 6)
}
{% endhighlight %}

The result is 
```
Today is 6/July/2017
```