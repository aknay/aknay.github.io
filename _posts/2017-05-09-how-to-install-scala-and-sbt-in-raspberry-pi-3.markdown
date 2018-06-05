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
$ wget https://downloads.lightbend.com/scala/2.12.4/scala-2.12.4.deb
$ sudo dpkg -i scala-2.12.4.deb
```

> Note: You an check Scala version with `$ scala -version`

### SBT
We will use package system to download
> Note: Make sure time/date of Raspberry Pi is updated. Otherwise, you will have `server certificate verification failed` error and cannot install SBT.

> Note: If you are having error such as `gpg: failed to start the dirmngr '/usr/bin/dirmngr': No such file or directory` while adding the keyserver, then you should first install `dirmngr` by using `sudo apt-get install dirmngr`

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

> Note: `Xmx256M` means we are allocating maximum 256MB to JVM. You can increase the memory to 512MB and so on. If you want to increase to 512MB then, the code will be `SBT_OPTS=-Xmx512M`.  

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
sbt new https://github.com/sbt/scala-seed.g8
```

When prompted for the project name, type `hello`. It should be something like this when it asks you to type in the project name. 

```sh
A minimal Scala project. 

name [Scala Seed Project]: hello

Template applied in ./hello
```

This will create a new project under a directory named `hello`.

```sh
$ cd hello
sbt
sbt:Hello> run
```
You will see
```sh
[info] Running example.Hello
hello
```

> Note: you can change `build.properties` inside hello folder  to `sbt.version=0.13.15` if fail to compile.


You can exit by typing `exit`.