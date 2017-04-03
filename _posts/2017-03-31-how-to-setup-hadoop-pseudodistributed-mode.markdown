---
layout: post
title:  "How to setup Hadoop for Pseudo-distributed Mode"
date:   2017-03-31 13:17:57 +0800
excerpt_separator: <!-- excerpt -->
---
The pseudo-distributed mode is a special case of fully distributed mode
in which the (single) host is localhost. Note that Hadoop daemon runs in a separate Java process. Therefore, we need to start daemons and we need to configure a couple settings until it can get it working. Let's see how it can be achieved

<!-- excerpt -->

### Related Topics

1. [Reference](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html){:target="_blank"}
2. [Reference Book](http://shop.oreilly.com/product/0636920033448.do){:target="_blank"}

### Hadoop Setup
You haven't setup your Hadoop system yet, you can check out my blog on [How to setup Hadoop -Standalone mode]({% post_url 2017-03-31-how-to-setup-hadoop-standalone-mode %}) for more info. 

### Configuring SSH
In pseudodistributed mode, we need SSH to log in to local host without having to enter a password. First, we install `SSH`.

```sh
$ sudo apt-get install ssh
```
To enable passwordless login to localhost,

```sh
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
Now you can test it by logging in to your localhost with SSH. It should not ask you for password now.

```sh
$ ssh localhost
```

### Settings
Now it's time to set some settings. Hope you still remember your Hadoop folder that you installed. Go to that Hadoop folder. Mine is `/home/aknay/HadoopWorkSpace/hadoop-2.8.0`. Now we will modify `core-site.xml` file.


```sh
$ nano etc/hadoop/core-site.xml
```

At the end of the file. we remove this empty block
```
<configuration>
</configuration>
```

 and replace with the following. Then save it.

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
Similarly, we will modify the config file for `hdfs-site.xml`.
```
$ nano etc/hadoop/hdfs-site.xml
```
with the following config.

```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```


### Formatting the HDFS file system
Before HDFS can be used for the first time, the file system must be formatted. This is
done by running the following command:
```sh
$ hdfs namenode -format
```

### HDFS Daemon

In order to start HDFS daemon, type
```sh
$ start-dfs.sh
```
Then go to `http://localhost:50070` to see web interface for the `NameNode`. Make sure you can access the web interface. Otherwise, you should check your configuration file and check that hdfs daemon is running.

> Note: You can stop hdfs daemon by typing `$ stop-dfs.sh`

### Create HDFS directories

Here, we create the HDFS directories required to execute

**General case:** 
```sh
$ bin/hdfs dfs -mkdir /user
$ bin/hdfs dfs -mkdir /user/<username>
```
**For my case:**
```sh
$ hdfs dfs -mkdir /user
$ hdfs dfs -mkdir /user/aknay
```

### Choose Your Path
1. MapReduce locally
2. MapReduce on YARN on a single node

If you want to run MapReduce locally, you can jump to the `Example` topic straight away. Otherwise, we need to setup a few more configuration for YARN.

### MapReduce on YARN

Here, We will configure for YARN for a single node. This file is haven't created yet but they provide a template for it. So we make a copy of it and then change the configuration. 

```sh
$ cp etc/hadoop/mapred-site.xml.template etc/hadoop/mapred-site.xml
$ nano etc/hadoop/mapred-site.xml
```
```
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

Same for `yarn-site.xml`. This file is already there. Just need to modifiy the configuration. 

```sh
$ nano etc/hadoop/yarn-site.xml
```

```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```
Now let's start YARN. 
```sh
$ start-yarn.sh
```

Now go to `http://localhost:8088`. Make sure you can access the web interface.

> Note:  You can stop the YARN by typing `$ stop-yarn.sh`


### Example
This example is same for us to run MapReduce locally or on YARN at a single node. If you are running locally, make sure `hdfs daemon` is running. If you are running YARN, make sure both `hdfs and yarn daemons` are running.

```sh
$ hdfs dfs -put etc/hadoop input
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.0.jar grep input output 'dfs[a-z.]+'
```
You can check the output by 

```
$ hdfs dfs -cat output/*
```

Your output might be like this.
```
6	dfs.audit.logger
4	dfs.class
3	dfs.logger
3	dfs.server.namenode.
2	dfs.audit.log.maxbackupindex
2	dfs.period
2	dfs.audit.log.maxfilesize
1	dfs.log
1	dfs.file
1	dfs.servers
1	dfsadmin
1	dfsmetrics.log
1	dfs.replication
```
You can also check at `output` directory.
```sh
$ hdfs dfs -ls output
```
If you are running on YARN, you can check the status on `http://localhost:8088`. The `FinalStatus` should be shown as `SUCCEEDED`.

> Note:  If you want to remove `output` folder or just want to run another round, you can remove it by `$ hdfs dfs -rm -r output`