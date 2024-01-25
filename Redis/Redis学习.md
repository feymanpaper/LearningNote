
### Redis特点
系统维度：可以归类为三高
1. 高性能：线程模型、网络 IO 模型、数据结构、持久化机制；
2. 高可用：主从复制、哨兵集群、Cluster 分片集群；
3. 高拓展：负载均衡

## 数据结构类型
### 常见数据类型和应用场景
#### String
常规计数, 分布式锁, 共享 Session 信息
### 数据结构
#### SDS
Redis 一共设计了 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64
其中sds表示simple dynarnic string，能存储2^n-1
这 5 种类型的主要**区别就在于，它们数据结构中的 len 和 alloc 成员变量的数据类型不同**。
扩容机制
![](Pasted%20image%2020240111001536.png)



## 待看
点赞计数的思考
https://tans.fun/archives/guan-yu-dian-zan-ji-shu-qi-de-yi-xie-si-kao
源码角度理解Redis底层原理
https://www.xiebruce.top/1820.html