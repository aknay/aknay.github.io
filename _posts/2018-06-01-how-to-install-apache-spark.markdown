---
layout: post
title:  "How to install Apache Spark for Raspberry Pi"
date:   2018-06-01 10:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
In this short note, we will install Apache Spark for Raspberry Pi.
<!-- excerpt -->

#### **Prerequisites**
A Raspberry Pi with network.
Tested on both Raspberry Pi 3 and 2. 

#### **Reference**
1. [Medium blog post](https://medium.com/@josemarcialportilla/installing-scala-and-spark-on-ubuntu-5665ee4b62b1){:target="_blank"}


#### **Overview**
There are a lot of blog posts about Apache Spark cluster using Raspberry Pi. This will be the initial start to install Apache Spark.  

#### **Update Raspberry Pi (Optional)**
This is just to update Raspberry Pi software if you haven't updated those for very long or you get a new Raspberry Pi.

You should/can skip `sudo rpi-update` if you are afraid of hardware incompatibility with your current hardware setup. 
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
$ sudo rpi-update
```

> Note: if you run `sudo rpi-update`, you should reboot the system by using `sudo reboot`

#### **Install Scala**
We need to install Scala programming language to use withing Apache Spark
```
$ sudo apt-get install scala
```

#### **Install Apache Spark**
You can always check the latest version from [here.](https://spark.apache.org/downloads.html){:target="_blank"}
```
$ wget http://www-us.apache.org/dist/spark/spark-2.3.0/spark-2.3.0-bin-hadoop2.7.tgz
$ tar -xvzf spark-2.3.0-bin-hadoop2.7.tgz
```
#### **Run Apache Spark**
Once you extracted the zip file, go to that folder and run spark-shell like this.
```
$ cd spark-2.3.0-bin-hadoop2.7/bin/
$ ./spark-shell
```
After you run the shell, you will be introduced by the Scala command line like the following.
```
scala>
```
Type in some Scala programming or just exit by Ctrl + c. We are done installing Apache Spark. Until then.