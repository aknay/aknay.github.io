---
layout: post
title:  "How to install Scala and SBT in Raspberry Pi 3"
date:   2017-05-09 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---

In this tutorial, we will install Scala and SBT in Raspberry Pi 3. And then we will tackle memory issue and finally, we will verify the setup with an example Scala program. Let's drive in.

<!-- excerpt -->

### Related Topics

1. [Reference](https://blog.knoldus.com/2016/09/04/scala-iot-first-basic-iot-application-using-scala-on-raspberrypi/){:target="_blank"}


### Java JDK 8
Scala needs JDK 8 and it is already installed on Raspberry Pi 3. If it is not installed due to some reasons, you can install by
`$ sudo apt-get install oracle-java8-jdk`
> Note: You can check Java version with `$ java -version`


### Scala  
You can check latest Scala from [here](https://www.scala-lang.org/download/){:target="_blank"}. 
```sh
$ wget https://downloads.lightbend.com/scala/2.12.2/scala-2.12.2.tgz
$ sudo mkdir /usr/lib/scala
$ sudo tar -xf scala-2.12.2.tgz -C /usr/lib/scala
$ rm scala-2.12.2.tgz
$ sudo ln -sf /usr/lib/scala/scala-2.12.2/bin/scala /bin/scala
$ sudo ln -sf /usr/lib/scala/scala-2.12.2/bin/scalac /bin/scalac
```

> Note: You an check Scala version with `$ scala -version`

### SBT
We will use package system to download

```sh
$ sudo apt-get install apt-transport-https
$ echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
$ sudo apt-get update
$ sudo apt-get install sbt
```
### Reducing Memory Usage
Raspberry Pi has limited memory and JVM will consume a lot of memory if we don't limit it. Therefore, we will modify the memory usage in SBT.

> Note: You can check where is you sbt file by `$ which sbt`

```sh
$ sudo nano /usr/bin/sbt
```

Add this line on top of the file
`SBT_OPTS=-Xmx256M` and save it.

You can check again with `$ sbt -d` and SBT info should show something like this below.
```
[process_args] java_version = '1.8'
# Executing command line:
java
-Xmx256M
-jar
/usr/share/sbt/bin/sbt-launch.jar

```

### Test with Sample Example

```
mkdir sbt_ws
cd sbt_ws
sbt new sbt/scala-seeed.g8
```

when prompted for the project type `hello`

> :exclamation: Note: If you cannot download due to network authenticity fail on Git after this command `sbt new sbt/scala-seeed.g8`, you can still clone the git by `git clone https://github.com/scala/scala-seed.g8.git`. Once it is cloned, go into `cd scala-seed.g8/src/main/g8` and then start typing `sbt` follow by `run`. You should see `hello`.

Then
```sh
cd hello
sbt
run
```
You will see
```sh
[info] Running example.Hello
hello
```

> Note: you can change `build.properties` inside hello folder  to `sbt.version=0.13.15` if fail to compile.


You can exit by typing `exit`.



