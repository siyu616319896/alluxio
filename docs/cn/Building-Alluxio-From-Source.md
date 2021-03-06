---
layout: global
title: 编译Alluxio代码主分支
nickname: 编译主分支
group: Resources
---

* 内容列表
{:toc}

该指南介绍如何从头编译Alluxio。

这部分内容的前提条件是你已安装[Java JDK 8或以上](Java-Setup.html)、[Maven 3.3.9或以上](Maven.html)以及[Thrift 0.9.3](Thrift.html) (可选)。

从Github上获取主分支并编译：

```bash
$ git clone git://github.com/alluxio/alluxio.git
$ cd alluxio
$ mvn install -DskipTests
```

若出现`java.lang.OutOfMemoryError: Java heap space`，请执行：

```bash
$ export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"
```

若需要编译一个特定的版本Alluxio，例如{{site.ALLUXIO_RELEASED_VERSION}}，先执行`cd alluxio`，接着执行`git checkout v{{site.ALLUXIO_RELEASED_VERSION}}`。

Maven编译环境将自动获取依赖，编译源码，运行单元测试，并进行打包。如果你是第一次编译该项目，下载依赖包可能需要一段时间，但以后的编译过程将会快很多。

一旦编译完成，执行以下命令启动Alluxio：

```bash
$ ./bin/alluxio format
$ ./bin/alluxio-start.sh local
```

若要确认Alluxio是否在运行，可以访问[http://localhost:19999](http://localhost:19999)，或者查看`alluxio/logs`目录下的日志文件，也可以执行下面的简单程序:

```bash
$ ./bin/alluxio runTests
```

若正确运行，应能看到类似以下输出结果：

```bash
/default_tests_files/BasicFile_STORE_SYNC_PERSIST has been removed
2015-10-20 23:02:54,403 INFO   (ClientBase.java:connect) - Alluxio client (version 1.0.0) is trying to connect with FileSystemMaster master @ localhost/127.0.0.1:19998
2015-10-20 23:02:54,422 INFO   (ClientBase.java:connect) - Client registered with FileSystemMaster master @ localhost/127.0.0.1:19998
2015-10-20 23:02:54,460 INFO   (BasicOperations.java:createFile) - createFile with fileId 1476395007 took 65 ms.
2015-10-20 23:02:54,557 INFO   (ClientBase.java:connect) - Alluxio client (version 1.0.0) is trying to connect with BlockMaster master @ localhost/127.0.0.1:19998
2015-10-20 23:02:54,558 INFO   (ClientBase.java:connect) - Client registered with BlockMaster master @ localhost/127.0.0.1:19998
2015-10-20 23:02:54,590 INFO   (WorkerClient.java:connect) - Connecting local worker @ /192.168.31.242:29998
2015-10-20 23:02:54,654 INFO   (FileUtils.java:createStorageDirPath) - Folder /Volumes/ramdisk/alluxioworker/6601007274872912185 was created!
2015-10-20 23:02:54,657 INFO   (LocalBlockOutStream.java:<init>) - LocalBlockOutStream created new file block, block path: /Volumes/ramdisk/alluxioworker/6601007274872912185/1459617792
2015-10-20 23:02:54,658 INFO   (WorkerClient.java:connect) - Connecting local worker @ /192.168.31.242:29998
2015-10-20 23:02:54,754 INFO   (BasicOperations.java:writeFile) - writeFile to file /default_tests_files/BasicFile_STORE_SYNC_PERSIST took 294 ms.
2015-10-20 23:02:54,803 INFO   (BasicOperations.java:readFile) - readFile file /default_tests_files/BasicFile_STORE_SYNC_PERSIST took 47 ms.
Passed the test!
```

执行以下命令停止Alluxio：

```bash
$ ./bin/alluxio-stop.sh local
```

## 单元测试

运行所有单元测试：

```bash
$ mvn test
```

在本地文件系统之外的存储层上运行所有单元测试：

```bash
$ mvn test [ -Dhadoop.version=x.x.x ] [ -P<under-storage-profile> ]
```

目前支持的`<under-storage-profile>`值包括：

```
Not Specified # [Default] Tests against local file system
swiftTest     # Tests against a simulated Swift cluster
hdfsTest      # Tests against HDFS minicluster
glusterfsTest # Tests against GlusterFS
s3Test        # Tests against Amazon S3 (requires a real S3 bucket)
ossTest       # Tests against Aliyun OSS (requires a real OSS bucket)
gcsTest       # Tests against Google Cloud Storage (requires a real GCS bucket)
```

若需将日志输出到STDOUT，将以下命令追加到`mvn`命令后：

```properties
-Dtest.output.redirect=false -Dalluxio.root.logger=DEBUG,CONSOLE
```

## 计算框架支持
自Alluxio 1.7开始，编译后位于`{{site.ALLUXIO_CLIENT_JAR_PATH}}`的Alluxio客户端jar包将适用于不同的计算框架（如：Spark、Flink，Presto等）。你不要在运行Maven编译的时候使用不同计算框架的配置文件。

## Hadoop分布式支持
要针对hadoop发行版本中某一个版本构建Alluxio，可以通过指定`<HADOOP_PROFILE>`和对应的hadoop.version`来运行如下命令：

```bash
$ mvn install -P<HADOOP_PROFILE> -Dhadoop.version=<HADOOP_VERSION> -DskipTests
```

`<HADOOP_VERSION>`可以被设置不同值。可用的Hadoop配置文件包括`hadoop-1`, `hadoop-2`, `hadoop-3`，涵盖主要的Hadoop版本1.x, 2.x和3.x。

### Apache
所有主要版本都来自Apache，所以所有Apache发行版都可以直接使用。

``properties
-Phadoop-1 -Dhadoop.version=1.0.4
-Phadoop-1 -Dhadoop.version=1.2.0
-Phadoop-2 -Dhadoop.version=2.2.0
-Phadoop-2 -Dhadoop.version=2.3.0
-Phadoop-2 -Dhadoop.version=2.4.1
-Phadoop-2 -Dhadoop.version=2.5.2
-Phadoop-2 -Dhadoop.version=2.6.5
-Phadoop-2 -Dhadoop.version=2.7.3
-Phadoop-2 -Dhadoop.version=2.8.0
-Phadoop-2 -Dhadoop.version=2.9.0
-Phadoop-3 -Dhadoop.version=3.0.0
```

### Cloudera
对于Cloudera发行版，使用该形式`$apacheRelease-cdh$cdhRelease`的版本号

```properties
-Phadoop-2 -Dhadoop.version=2.3.0-cdh5.1.0
-Phadoop-2 -Dhadoop.version=2.0.0-cdh4.7.0
```

### MapR

对于MapR发行版，其值为

```properties
-Phadoop-2 -Dhadoop.version=2.7.0-mapr-1607
-Phadoop-2 -Dhadoop.version=2.7.0-mapr-1602
-Phadoop-2 -Dhadoop.version=2.7.0-mapr-1506
-Phadoop-2 -Dhadoop.version=2.3.0-mapr-4.0.0-FCS
```

### Hortonworks

对于Hortonworks发行版，使用`$apacheRelease.$hortonRelease`形式的版本号

```properties
-Phadoop-2 -Dhadoop.version=2.1.0.2.0.5.0-67
-Phadoop-2 -Dhadoop.version=2.2.0.2.1.0.0-92
-Phadoop-2 -Dhadoop.version=2.4.0.2.1.3.0-563
```

## 系统设置

有时为了在本地通过单元测试，需要进行些系统设置，常用的一个设置项为ulimit。

### Mac

为增加允许的文件以及进程数目，执行以下命令：

```bash
$ sudo launchctl limit maxfiles 32768 32768
$ sudo launchctl limit maxproc 32768 32768
```

推荐将Alluxio本地资源从Spotlight索引中移除，否则你的Mac机器在单元测试中可能会尝试重复对文件系统构建索引从而挂起。若要进行移除，进入`System Preferences > Spotlight > Privacy`，点击`+`按钮，浏览Alluxio的本地资源目录，并点击`Choose`将其添加到排除列表。
