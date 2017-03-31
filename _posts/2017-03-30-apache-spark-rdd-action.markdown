---
layout: post
title:  "Apache Spark- RDD Actions"
date:   2017-03-30 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---

RDD actions are RDD operations which don't generate another RDD. Instead, RDD actions return a value of any types (such as List()) but not RDD[T]. And RDD actions are just like any other RDD operations. They are lazy; meaning they don't compute right away, only when an action requires to return values. Let's see how it's done.   


<!-- excerpt -->

### Related Topics

1. [Reference Book](http://shop.oreilly.com/product/0636920028512.do){:target="_blank"}

### Functions that will be covered ###
+ collect()
+ count()
+ countByValue()
+ countByKey()
+ take(n)
+ takeOrdered(n, [ordering])
+ takeSample(withReplacement, num, [seed])
+ reduce(func)

### Collect Function

We will first create a standalone program. More info can be found [here](http://spark.apache.org/docs/latest/quick-start.html#self-contained-applications){:target="_blank"} to create a standalone program.

**Purpose:** `Return all elements from the RDD`

This is pretty simple. You just collect the elements which are defined in `Rdd`. 

```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD

object Spark {
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("Simple Application").setMaster("local[2]").set("spark.executor.memory", "1g");
    val sc = new SparkContext(conf)

    val rdd: RDD[Int] = sc.parallelize(List(1, 2, 3))
    val result: String = rdd.collect().mkString(",")
    println("the result is: " + result)

    sc.stop()
  }
}

```
> the result is: 1,2,3

---

### count Function

This time, the fragment of code will be shown from now on.

**Purpose:** `Number of elements in the RDD.`

```scala
    val rdd: RDD[Int] = sc.parallelize(List(1, 2, 3))
    val result: Long = rdd.count()
    println("the result is: " + result)
```
> the result is : 3

---

### countByValue Function
**Purpose:** `Number of times each element occurs in the RDD.`


```scala
    val rdd: RDD[Int] = sc.parallelize(List(1, 2, 3, 3))
    val result: collection.Map[Int, Long] = rdd.countByValue()
    println("the result is: " + result)
```
> the result is: Map(2 -> 1, 1 -> 1, 3 -> 2)

---

### countByKey Function

**Purpose:** `Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key.`

```scala
    val rdd: RDD[(Int, Int)] = sc.parallelize(List(1 -> 2, 3 -> 4, 3 -> 6))
    val result: collection.Map[Int, Long] = rdd.countByKey()
    println("the result is: " + result)
```

> the result is: Map(1 -> 1, 3 -> 2)

---


### take Function

**Signature**: `take(n)`

**Purpose:** `Return an array with the first n elements of the dataset.`

```scala
    val rdd: RDD[Int] = sc.parallelize(List(1, 2, 3, 3))
    val result: Array[Int] =  rdd.take(2)
    println("the result is: " + result.mkString(","))
```
> the result is: 1,2

---

### takeOrdered Function

**Signature**: `takeOrdered(n, [ordering])`

**Purpose:** Return the first n elements of the RDD using either their natural order or a custom comparator.`

```scala
    val rdd: RDD[Int] = sc.parallelize(List(1, 2, 3, 3))

    val firstResult: Array[Int] =  rdd.takeOrdered(2)
    println("the first result is: " + firstResult.mkString(","))

    val secondResult: Array[Int] =  rdd.takeOrdered(2)(Ordering[Int].reverse)
    println("the second result is: " + secondResult.mkString(","))
```
> the first result is: 1,2

> the second result is: 3,3

---

### takeSample Function

**Signature**: `takeSample(withReplacement, num, [seed])`

**Purpose:** Return an array with a random sample of num elements of the dataset, with or without replacement, optionally pre-specifying a random number generator seed.`

```scala
    val rdd: RDD[Int] = sc.parallelize(List(1, 2, 3, 3))
    val result: Array[Int] =  rdd.takeSample(withReplacement = false, 2)
    println("the first result is: " + result.mkString(","))
```
Note: The results are nondeterministic.
> the result is: 2,3

---

### reduce Function

**Signature**: `reduce(func)`

**Purpose:** Aggregate the elements of the dataset using a function func (which takes two arguments and returns one). The function should be commutative and associative so that it can be computed correctly in parallel.`

```scala
    val rdd: RDD[Int] = sc.parallelize(List(1, 2, 3, 3))
    val result: Int =  rdd.reduce((a, b) => a + b)
    println("the result is: " + result)
```

> the result is: 9

---




