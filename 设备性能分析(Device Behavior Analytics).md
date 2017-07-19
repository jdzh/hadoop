---
layout: post
title:  设备性能分析
date:   2017-07-19
categories: hdfs
tags: analytics datanode disk jmx metric
---

__简介：
设备性能分析通过标注出集群中慢的datanode和磁盘读写，来提升datanode性能问题的可诊断性。__  

【[社区原文](https://community.hortonworks.com/articles/109344/device-behavior-analytics.html)】  
设备性能分析(Device Behavior Analytics)  
正文：  
设备性能分析能够让管理员识别出那些拖慢集群性能的节点或磁盘。找出并修复这些节点能够提升整个集群的吞吐量。这些指标将在HDP-2.6.1及以后版本中集成进去。有个性能分析部分--慢节点检测和慢磁盘检测。

__慢节点检测：__  

datanode将收集统计其对等点正常运行过程中写入管道的延迟信息。这些延迟统计数据用于检测对等点之间的异常值。慢检测只在10个或更多的对等点才起作用。Namenode维护缓慢的对等节点列表，管理员可以通过JMX读取它，datanode也通过自身的对等点发布其平均写延迟到JMX。  

__慢磁盘检测：__ 

每个datanode会从他所有的磁盘中收集I/O统计信息。我们可以修改配置文件中I/O事件的百分比，来对影响I/O性能的点进行采样。磁盘性能检测只在5块及以上磁盘时才能执行。慢磁盘信息可以通过datanode的JMX中获得。namenode也会通过namenode的JMX向集群发布慢磁盘信息。  

__开启设备性能分析：__  

开启设备性能分析需要做如下配置：
``` 
* dfs.datanode.peer.stats.enabled
    #设置为true
* dfs.datanode.fileio.profiling.sampling.percentage  
    #设置1到100之间的值，这个值是采样点影响IO性能的值，当设置为100时，将采样所有值。对大量磁盘IO事件进行抽样可能会产生一点小的性能影响。
* dfs.datanode.outliers.report.interval  
    #这个配置是你控制datanode向namenode通过心跳发送延迟统计信息频率的值，默认为30min。
```
上述配置需要添加到hdfs-site.xml中。如果是基于ambari安装的，将该配置加入到定制的hdfs-site.xml中（界面配置）。


