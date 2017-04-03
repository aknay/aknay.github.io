---
layout: post
title:  "How to setup Hadoop for Standalone Mode"
date:   2017-03-31 10:17:57 +0800
excerpt_separator: <!-- excerpt -->
---

In this tutorial, we will setup Hadoop for standalone mode. Standalone
mode is suitable for running MapReduce programs during development, since it is
easy to test and debug them. Note that the default properties are
already set for standalone mode and there are no daemons to run. Let's look at it, shall we?

<!-- excerpt -->

### Related Topics

1. [Reference](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html){:target="_blank"}
2. [Reference Book](http://shop.oreilly.com/product/0636920033448.do){:target="_blank"}

### Download 
You can check latest Hadoop from [here](http://hadoop.apache.org/releases.html){:target="_blank"}. Please take note of the version that you wish to install. You will need it later.

```sh
$ wget http://www-us.apache.org/dist/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz
$ tar -xvzf hadoop-2.8.0.tar.gz
```
### Set JAVA_HOME
Please make sure Java is installed. You can download from [here](http://www.oracle.com/technetwork/java/javase/downloads/index.html){:target="_blank"} if you haven't done it so.

You can find your java home by typing
```sh
$ echo $JAVA_HOME
```
mine is `/usr/lib/jvm/java-8-oracle`. Now let set `JAVA_HOME` in this file `hadoop-env.sh`.


```sh
$ nano etc/hadoop/hadoop-env.sh
```

find the following line in `hadoop-env.sh`

```
# The java implementation to use.
export JAVA_HOME=${JAVA_HOME}
```

and replace with your java home path.
I modified mine like this and save it.

```
# The java implementation to use.
export JAVA_HOME=/usr/lib/jvm/java-8-oracle
```

### Environment variable
You should set the environment variable for Hadoop. It's much easier to execute Hadoop from anywhere.

**General case**

```sh
$ export HADOOP_HOME=<Your Hadoop Installation Folder That You Just Unzipped>
$ export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

**For my case**

Because I installed Hadoop 2.8.0 under `/home/aknay/HadoopWorkSpace`. So I set my environment variable as this way. You can use `pwd` command to find the full path from unzipped Hadoop folder.

```sh
$ export HADOOP_HOME=/home/aknay/HadoopWorkSpace/hadoop-2.8.0
$ export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
You can check whether the environment variable are set correctly, you can type in 
```sh
$ hadoop version
```
And you will see some message like this
```
Hadoop 2.8.0
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r 91f2b7a13d1e97be65db92ddabc627cc29ac0009
Compiled by jdu on 2017-03-17T04:12Z
Compiled with protoc 2.5.0
From source with checksum 60125541c2b3e266cbf3becc5bda666
This command was run using /home/aknay/HadoopWorkSpace/hadoop-2.8.0/share/hadoop/common/hadoop-common-2.8.0.jar
```
### Test Standalone Mode
There is no additional setup for standalone mode. We can just test an example with a few commands.

First, create a folder called `input` under `hadoop` folder. The `hadoop` folder for me is `/home/aknay/HadoopWorkSpace/hadoop-2.8.0`
```sh
$ mkdir input
```
Then follow by

```sh
$ cp etc/hadoop/*.xml input
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.0.jar grep input output 'dfs[a-z.]+'
```
After the last command, you will see a long message. The above command is to find and displays every match of the given regular expression from `input` directory. The output is written to the `output` directory. So let's check the result.

```sh
$ cat output/*
```

You might see some result like this.
```
1   dfsadmin
```

You can also view the generated files at the `output` folder

```sh
$ ls -ls output/
``` 
You should see `_SUCCESS` file inside the `output` folder. And that's it.