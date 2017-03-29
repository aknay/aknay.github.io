---
layout: post
title:  "Scala Collections - map function"
date:   2017-03-29 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
The purpose of map function is to build a new collection by applying a function to all elements of this concurrent map. Let's explore with more examples.

<!-- excerpt -->
### Apply To Every Element On The List

Every item is multiplied by 2 since we supply `x => x * 2` anonymous function. 

```scala
    val result: List[Int] = List(1, 2, 3) map (x => x * 2)
    println("the result is: " + result)
```
> the result is: List(2, 4, 6)

In this example, we want to know the employers who have more than 2500 salary income.

```scala
    val result: Map[String, Boolean] = Map("Albert" -> 1000, "Bob" -> 2000, "Carl" -> 3000).map {
      case (a, b) => (a, b > 2500)
    }
    println("the result is: " + result)
```
> the result is: Map(Albert -> false, Bob -> false, Carl -> true)

---

### As Filter

It can also be used as a filter. Here, `pattern matching` is used. This example keeps the values which is divisible by 2.

```scala
    val result: List[Int] = List(1, 2, 3, 4, 5, 6).map {
      case x if (x % 2 == 0) => x
      case _ => 0
    }
    println("the result is: " + result)
```
> the result is: List(0, 2, 0, 4, 0, 6)

---

### Get different type of List

We can also convert to another type (in this case, `List[String]` to `List[Int]`). We can even sum it up to see how many chars in the list.

```scala
    val result: List[Int] = List("a", "abcd", "abc").map {
      x => x.length
    }
    println("the result is: " + result)
```
> the result is: List(1, 4, 3)

### Replace map with `for comprehension`

The same result of above example can be achieved using `for comprehension`. Take note of `for` and `yield` keyword.

```scala
    val result: List[Int] = for (s <- List("a", "abcd", "abc")) yield s.length
    println("the result is: " + result)
```
> the result is: List(1, 4, 3)

---