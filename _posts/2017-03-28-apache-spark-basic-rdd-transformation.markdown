---
layout: post
title:  "Apache Spark- Basic RDD Transformation"
date:   2017-03-28 21:17:57 +0800
excerpt_separator: <!-- excerpt -->
---

RDD transformation is to create a new dataset from an existing one. For example, map is a transformation that passes each dataset element through a function and returns a new RDD representing the results. Let's look at more examples. 

<!-- excerpt -->
### Map Function

We will first create a standalone program. More info can be found [here](http://spark.apache.org/docs/latest/quick-start.html#self-contained-applications){:target="_blank"}.

**Purpose:** `Apply a function to each element in the RDD and return an RDD of the result.`

```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD

object SimpleApp {
  def main(args: Array[String]) {
    val conf = new SparkConf().setAppName("Simple Application").setMaster("local[2]").set("spark.executor.memory", "1g");
    val sc = new SparkContext(conf)

    val input: RDD[Int] = sc.parallelize(List(1, 2, 3, 3))
    val rdd: RDD[Int] = input.map(x => x + 1)
    print("the result is : " + rdd.collect().mkString(","))

    sc.stop()
  }
}

```
> the result is : 2,3,4,4

---

### FlapMap Function

This time, the fragment of code will be shown from now on.

**Purpose:** `Apply a function to each element in the RDD and return an RDD of the contents of the iterators returned. Often used to extract words.`

```scala
    val input: RDD[List[Int]] = sc.parallelize(List(List(1, 2), List(6, 7)))
    val rdd: RDD[Int] = input.flatMap(sublist => sublist.map(x => x + 1))
    print("the result is : " + rdd.collect().mkString(","))
```
> the result is : 2,3,7,8

---

### Filter Function
**Purpose:** `Return an RDD consisting of only elements that pass the condition passed to filter()`

```scala
    val input: RDD[Int] = sc.parallelize(List(1, 2, 3, 3))
    val rdd: RDD[Int] = input.filter(x => x != 3)
    print("the result is : " + rdd.collect().mkString(","))
```
> the result is : 1,2

---

### Distinct Function

**Purpose:** `Remove duplicates.`

```scala
    val input: RDD[Int] = sc.parallelize(List(1, 2, 3, 3))
    val rdd: RDD[Int] = input.distinct()
    print("the result is : " + rdd.collect().mkString(","))
```
> the result is : 2,1,3

---

### Sample Function
**Purpose:** `Sample a fraction of the data, with or without replacement, using a given random number generator seed.`

```scala
    val input: RDD[Int] = sc.parallelize(1 to 10)
    val rdd: RDD[Int] = input.sample(withReplacement = false, 0.5)
    print("the result is : " + rdd.collect().mkString(","))
```
> Nondeterministic

---