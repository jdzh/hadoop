---
layout: post
title:  通过pyspark的流计算进行单词统计的例子
date:   2017-07-09 17:00:00
categories: Spark
tags: Spark pyspark sparkstreaming wordcount
---

__简介：这是一个以本地文件系统为源的WordCount示例文章__

【[社区原文](https://community.hortonworks.com/articles/81359/pyspark-streaming-wordcount-example.html)】  
通过pyspark的流计算进行单词统计的例子(Pyspark Streaming Wordcount Example)  
正文

__通过pyspark的流计算进行单词统计的例子__

具体内容如下
* 本地文件系统作为源
* 使用reduceByKey计算计数并将其存储在临时表中
* 通过SQL查询运行计数  

__设置：定义设置StreamingContext的方法__  
该方法是创建一个StreamingContext，并进行必要配置。  
但是它不能启动StreamingContext，它只是将你对该应用的一些配置参数放到一个方法中去。
```
from pyspark.streaming import StreamingContext
batchIntervalSeconds = 10 
 
def creatingFunc():
  ssc = StreamingContext(sc, batchIntervalSeconds)
  # Set each DStreams in this context to remember RDDs it generated in the last given duration.
  # DStreams remember RDDs only for a limited duration of time and releases them for garbage
  # collection. This method allows the developer to specify how long to remember the RDDs (
  # if the developer wishes to query old data outside the DStream computation).
  ssc.remember(60)
  
  lines = ssc.textFileStream("YOUR_S3_PATH_HERE")
  lines.pprint()
 
  words = lines.flatMap(lambda line: line.split(","))
  pairs = words.map(lambda word: (word, 1))
  wordCounts = pairs.reduceByKey(lambda x, y: x + y)
 
  def process(time, rdd):
    df = sqlContext.createDataFrame(rdd)
    df.registerTempTable("myCounts")
  
  wordCounts.foreachRDD(process)
  
  return ssc
```
__创建streaming context__  
如果您不希望该应用程序从检查点重新启动，请确保删除检查点目录

```
checkpointDir = "/LOCAL_DIR/DATA/"
ssc = StreamingContext.getActiveOrCreate(checkpointDir, creatingFunc)
```
__启动streaming context__
```
# This line starts the streaming context in the background.
ssc.start()
 
# This ensures the cell is put on hold until the background streaming thread is started properly.
ssc.awaitTerminationOrTimeout(batchIntervalSeconds * 2)
```
__终端输出：__

> 
> ------------------------------------------- Time: 2015-10-07 20:57:00 -------------------------------------------  
> hello,world hello,world hello,world hello,world hello,world hello,world hello,world hello,world hello,world hello,world ...  
> ------------------------------------------- Time: 2015-10-07 20:57:10 -------------------------------------------  
> hello,world hello,world hello,world hello,world hello,world hello,world hello,world hello,world hello,world hello,world ...
>   

__交互查询__

```
%sql select * from myCounts
```
__终端输出：__
> world  
> hello

__关闭streaming context__
```
ssc.stop(stopSparkContext=False)
```

__停掉所有jvm中的streaming context__  
当你丢失streaming contexts句柄的时候，这一步是非常必要的。比如说，你通过notebook运行程序，退出当前页面，而流计算仍然在运行。在这种情况下，你丢掉了streaming contexts的句柄连接但是他仍在后台运行。如果你想停掉它，那么你需要直接连接jvm然后停掉。  
```
if( not sc._jvm.StreamingContext.getActive().isEmpty() ): 
	sc._jvm.StreamingContext.getActive().get().stop(False)
```
