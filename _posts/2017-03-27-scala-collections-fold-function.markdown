---
layout: post
title:  "Scala collections- Fold"
date:   2017-03-27 21:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
Official Definition: Folds the elements of this mutable indexed sequence using the specified associative binary operator. Let's dive deep into this using examples.

<!-- excerpt -->

### Note
`fold()` needs initial value whereas `reduce` doesn't need an initial value.

### For summation
``` scala
    val result: Int = List(1,2,3).fold(0)(_+_)
    println("the result is: "+ result)
```
> the result is: 6

The same thing can be done with this `List(1,2,3).sum`.

The initial value is `0` because of `fold(0)`. Then  the initial value `0` is added with first element `1`. Therefore, `0 + 1 = 1`. This resultant `1` is added with the second element `2`. So `1 + 2 = 3`. Then `3` is added to the last element `3` and again `3 + 3 = 6`. The final result `6` is returned. 

You can also find the difference in this way.

```scala
    val result: Int = List(1,2,3).fold(5)(_-_)
    println("the result is: "+ result)
```
> the result is: -1

The inital value is `5` because of `fold(5)`. Then  the initail value `5` is subtracted with first element `1`. Therefore, `5 - 1 = 4`. This resultant `4` is subtracted with the second element `2`. So `4 - 2 = 2`. Then `2` is subtracted with the last element `3` and again `2 - 3 = -1`. The final result `-1` is returned. 

### Get Largest Element

```scala
    val result = List(1,2,3).fold(0)((a,b) => {
      if (a > b) a
      else b
    })

    println("the result is: "+ result)
```
> the result is: 3

The same result can be achieved with this `List(1,2,3).fold(0)(_ max _)`.

### Get Smallest Element
Take note of change in sign `<`. We need to include initial value `0` while we are evaluating the expression. Due to initial value `0`, the smallest value is `0` among (0,1,2,3) 

```scala
    val result = List(1,2,3).fold(0)((a,b) => {
      if (a < b) a
      else b
    })

    println("the result is: "+ result)
```
> the result is: 0

The same result can be achieved with this `List(1,2,3).fold(0)(_ min _)`.

### Get Longest String in the List

```scala
    val result: String = List("abc", "abcdef").fold("ab")((a, b) => {
      if (a.length > b.length) a
      else b
    })

    println("the result is: "+ result)
```
> the result is: abcdef

### Find Largest value in the Map List

Here `a._2 > b._2` means we are comparing `Int` value of the `Map`. The `a._1` can be referred to the first value and `a._2` can be referred as second value of Map.
For example: In `"Albert"-> 1000`  Map, the `a._1` is `Albert` and the `a._2` is `1000`  

```scala
    val result: (String, Int) = List("Albert"-> 1000,"Bob" -> 2000 ,"Carl" -> 3000).fold(("",0))((a, b) => {
      if (a._2 > b._2) a
      else b
    })
    println("the result is: "+ result)
```
> the result is: (Carl,3000)

### Use Match with Fold
Here, just want to show that `match` can be used with `fold`. In this case, we add all the values which is divisible by 4. Only 4 and 8 are divisiable 4 among the number 1 to 10. So `4 + 8` gives `12`. 

```scala
    val result: Int = (1 to 10).fold(0)((a, b) =>
      b % 4 match {
        case 0 => a + b
        case _ => a
      })

    println("the result is: " + result)
```

> the result is: 12