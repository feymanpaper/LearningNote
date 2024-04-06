## 大数据基础
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
spark是单纯的计算框架本身不具备存储能#力, 取代hadoop的mapreduce计算框架
HDFS+Spark
#### Flink
![](Pasted%20image%2020240406001503.png)
## Spark
### Spark概述
spark运行速度快: 内存计算，循环数据流, DAG有向无环图流水线优化
完整的软件栈
![](Pasted%20image%2020240406102154.png)
运行模式多样
可以单机也可以集群运行
![](Pasted%20image%2020240406102248.png)
### Spark生态系统
![](Pasted%20image%2020240406103126.png)
传统是不同应用类型用不同的技术进行管理，hadoop批处理，storm流处理
资源利用不充分，不同技术需要进行格式转换无法无缝共享，维护成本高
spark一个软件栈能满足所有的需求
![](Pasted%20image%2020240406103555.png)
![](Pasted%20image%2020240406103946.png)
![](Pasted%20image%2020240406104309.png)
一主多从，主节点是driver program, 从节点是worker node
![](Pasted%20image%2020240406104615.png)
集群资源管理器，是由自带/yarn/mesos来管理cpu，内存，带宽
### Spark运行流程
见ppt
### RDD运行原理
见ppt