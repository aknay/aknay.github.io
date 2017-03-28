---
layout: post
title:  "Scala Collections - flatMap"
date:   2017-03-28 21:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
flatMap is a frequently used combinator that combines mapping and flattening. flatMap takes a function that works on the nested lists and then concatenates the results back together. It can be very confusing at first. Let's break it down with more examples.

<!-- excerpt -->
### With A Single List

flapMap can be used with a single list. Take note of `List(x*2)` signature. every item is multiplied by 2. After that, all lists are flattened as one list. 

```scala
    val result: List[Int] = List(1, 2, 3, 4, 5, 6).flatMap(x => List(x * 2))
    val resultAsString: String = result.mkString(",")
    println("the result is: " + resultAsString)
```
> the result is: 2,4,6,8,10,12

---

### With Multiple Lists

flatMap can also be used for multiple lists. Then we use `flatMap` then `map` pattern. Similar to the first flatMap with the single list, every item is multiplied by 2 and all lists are flattened as one list. 

```scala
    val result: List[Int] = List(List(1, 2, 3), List(3, 4, 5)).flatMap(x => x.map(i => i * 2))
    val resultAsString: String = result.mkString(",")
    println("the result is: " + resultAsString)
```
> the result is: 2,4,6,6,8,10

---

### flatMap as Filter

flatMap can also be used as a filter. Here, `pattern matching` and `Option` are used

```scala
    val result: List[Int] = List(1, 2, 3, 4, 5, 6).flatMap {
      case x if x % 2 == 0 => Some(x)
      case _ => None
    }
    val resultAsString: String = result.mkString(",")
    println("the result is: " + resultAsString)
```
> the result is: 2,4,6

---

### flatMap to Retrieve Known Data

To retrieve known data or value, flapMap can be used too. Similar to `flatMap as filter`, `pattern matching` and `Option` are used. Here, we extract `1`, `2` and `4` which can be parsed by using `Interger.parseInt`. Those that cannot be parsed will return as `None`.

```scala
    val result: List[Int] = List("1", "2", "three", "4", "five", "scala").flatMap { x =>
      try {
        Some(Integer.parseInt(x.trim))
      } catch {
        case e: Exception => None
      }
    }
    val resultAsString: String = result.mkString(",")
    println("the result is: " + resultAsString)
```
> the result is: 1,2,4

flatMap can also be used as a filter on the list with multiple types. 

```scala
    val result: List[String] = List("no", 1, true, "life").flatMap {
      case s: String => Some(s)
      case _ => None
    }
    val resultAsString: String = result.mkString(",")
    println("the result is: " + resultAsString)

```
> the result is: no,life

---