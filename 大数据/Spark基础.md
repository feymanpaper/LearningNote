### 大数据计算模式
批处理计算
MapReduce, Spark
流计算
实时处理, 给出实时响应,否则结果会失去商业价值
Storm, Flume
图计算
Google Pregel, GraphX
查询分析
Hive, Cassandra
![](Pasted%20image%2020240405233324.png)
### 大数据关键技术
分布式存储，分布式处理
![](Pasted%20image%2020240405232823.png)
#### Hadoop
![](Pasted%20image%2020240405233938.png)
mapreduce
![](Pasted%20image%2020240405234408.png)
#### Spark
hadoop缺点: mapreduce表达能力有限, 磁盘IO消耗大, 延迟高
spark基于内存计算速度快, DAG任务调度执行机制, 流水线优化
spark是单纯的计算框架本身不具备存储能力, 取代hadoop的mapreduce计算框架
HDFS+Spark
#### Flink
![](Pasted%20image%2020240406001503.png)