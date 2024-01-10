## 数据结构类型
### 常见数据结构和应用场景
#### SDS
Redis 一共设计了 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64
其中sds表示simple dynarnic string，能存储2^n-1
这 5 种类型的主要**区别就在于，它们数据结构中的 len 和 alloc 成员变量的数据类型不同**。
扩容机制
![](Pasted%20image%2020240111001536.png)


## 其他
### 点赞计数的思考
https://tans.fun/archives/guan-yu-dian-zan-ji-shu-qi-de-yi-xie-si-kao