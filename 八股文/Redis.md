### 如何保证Redis和DB一致性
个人思考:
#### Cache Aside
适合读多写少的场景, 最终一致性
**先更新数据库，再删缓存**
保证两个操作都能执行成功
- 重试机制。
- 订阅 MySQL binlog，再操作缓存。
#### Write Through
适合读多写少的场景,最终一致性
https://www.bilibili.com/video/BV1Z14y1D74Y?p=18&vd_source=108d23f95683578313bdaf5d938b5b3d
数据库+缓存双写+冷热分离(过期时间+自动expireTime过期)+load db&write cache+缓存惊群解决方案(随机过期时间)+缓存穿透解决方案(db load null->cache empty data)+数据库&缓存一致性方案(写和读, 分布式锁+读double check)+分布式锁串行转并发方案(读操作的分布式锁, tryLock, 自动超时)

适合写多，最终一致性 的场景，如高并发扣减库存
https://www.bilibili.com/video/BV1V64y1e7pH/?spm_id_from=333.976.header_right.fav_list.click&vd_source=108d23f95683578313bdaf5d938b5b3d
![](Pasted%20image%2020240108132715.png)
insert顺序写日志，日志异步同步到数据库中，由于事务和分布式锁的存在，缓存和日志的数据一定是一致的. 

为什么需要分布式锁？因为有可能对一个DB存在缓存无的数据写，如果此时有线程来读，会把DB的旧值读进来
#### Write Behind/Back方案
适合写多的，一致性没那么强(最终一致性)的场景，如点赞
https://www.cnblogs.com/crazymakercircle/p/17168230.html
**用的策略是： 异步写入+批量写入**
**异步写入**
同时数据库的写入我们也做了全面的异步化处理，保证了数据库能以合理的速率处理写入请求。
**批量写入（聚合写入）**
针对写流量，为了保证数据写入性能，B站在写入【点赞数】数据的时候，在内存中做了部分聚合写入，比如聚合10s内的点赞数，一次性写入。
如此可大量减少数据库的IO次数。
![](Pasted%20image%2020240108133312.png)

PS:消息队列中消息的异步写入磁盘、MySQL 的 InnoDB Buffer Pool 机制都用到了这种策略
