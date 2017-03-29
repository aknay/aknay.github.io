---
layout: post
title:  "Apache Spark- Two RDD Transformation"
date:   2017-03-29 20:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
Two RDD transformation is to create a new dataset by using two datasets. There are only a few functions to support this transformation. So let's take a look.

<!-- excerpt -->

### Related Topics
1. [Basic Rdd Transformation]({% post_url 2017-03-28-apache-spark-basic-rdd-transformation %})

2. [Reference Book](http://shop.oreilly.com/product/0636920028512.do){:target="_blank"}

### Union Function

We will first create a standalone program. More info can be found [here](http://spark.apache.org/docs/latest/quick-start.html#self-contained-applications){:target="_blank"} to create a standalone program.

**Purpose:** `Produce an RDD containing elements from both RDDs.`

```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD

object Spark {
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("Simple Application").setMaster("local[2]").set("spark.executor.memory", "1g");
    val sc = new SparkContext(conf)

    val firstRdd: RDD[Int] = sc.parallelize(List(1, 2, 3))
    val secondRdd: RDD[Int] = sc.parallelize(List(3, 4, 5))
    val result: Array[Int] = firstRdd.union(secondRdd).collect()
    print("the result is : " + result.mkString(","))

    sc.stop()
  }
}

```
> the result is : 1,2,3,3,4,5

---

### Intersection Function

This time, the fragment of code will be shown from now on.

**Purpose:** `RDD containing only elements found in both RDDs.`

```scala
    val firstRdd: RDD[Int] = sc.parallelize(List(1, 2, 3))
    val secondRdd: RDD[Int] = sc.parallelize(List(3, 4, 5))
    val result: Array[Int] = firstRdd.intersection(secondRdd).collect()
    print("the result is : " + result.mkString(","))
```
> the result is : 3

---

### Subtract Function
**Purpose:** `Remove the contents of one RDD (e.g.,remove training data).`

Here, firstRdd has `1,2,3` and secondRdd has `3,4,5`. When `subtract` function is called, the elements which are included on both Rdds are removed from firstRdd. Since `3` is included on both Rdds, `3` is removed from firstRdd. Therefore, only `1,2` are left on firstRdd.

```scala
    val firstRdd: RDD[Int] = sc.parallelize(List(1, 2, 3))
    val secondRdd: RDD[Int] = sc.parallelize(List(3, 4, 5))
    val result: Array[Int] = firstRdd.subtract(secondRdd).collect()
    print("the result is : " + result.mkString(","))
```
> the result is : 2,1

---

### Cartesian Function

**Purpose:** `Cartesian product with the other RDD.`

In this example, firstRdd has `1,2,3` and secondRdd has `3,4,5` . The first element of firstRdd `1` is paired with the first element of secondRdd `3`. The resultant of this action generates `(1,3)`. Next, the first element of firstRdd `1` is paired with the second element of secondRdd `4` which generates `(1,4)`. And same pairing for the rest of the elements. 

```scala
    val firstRdd: RDD[Int] = sc.parallelize(List(1, 2, 3))
    val secondRdd: RDD[Int] = sc.parallelize(List(3, 4, 5))
    val result: Array[(Int, Int)] = firstRdd.cartesian(secondRdd).collect()
    print("the result is : " + result.mkString(","))
```
> the result is : (1,3),(1,4),(1,5),(2,3),(3,3),(2,4),(2,5),(3,4),(3,5)

---