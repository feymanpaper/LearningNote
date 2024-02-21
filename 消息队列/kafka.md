美团-消息队列设计精要
https://tech.meituan.com/2016/07/01/mq-design.html

![](Pasted%20image%2020240125150414.png)

如何保证消息顺序性？
1.单个partition，单个消费者
2.维护状态机
http://timd.cn/b/1/#8-maingo
在内存维护一个hashmap，key是消息的id，需全局唯一，value是cacheitem，可以携带顺序key（比如时间）。在cacheitem里面记录乱序的消息。每有一个消息进来，排序（可以先按时间排再按状态机顺序排），然后用状态机流转cacheitem里面的乱序消息，更新为当前的状态。状态流转完进入终止需要删除这个消息。或者采用一个定时任务定期删除过期的消息，防止内存一直增长
3.版本号+重发机制