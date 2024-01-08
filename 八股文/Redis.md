### 如何保证Redis和DB一致性
个人思考:
#### Write Through方案
适合读多写少的场景
https://www.bilibili.com/video/BV1Z14y1D74Y?p=18&vd_source=108d23f95683578313bdaf5d938b5b3d
数据库+缓存双写+冷热分离(过期时间+自动expireTime过期)+load db&write cache+缓存惊群解决方案(随机过期时间)+缓存穿透解决方案(db load null->cache empty data)+数据库&缓存一致性方案(写和读, 分布式锁+读double check)+分布式锁串行转并发方案(读操作的分布式锁, tryLock, 自动超时)

适合写多，一致性强的场景，如高并发扣减库存
https://www.bilibili.com/video/BV1V64y1e7pH/?spm_id_from=333.976.header_right.fav_list.click&vd_source=108d23f95683578313bdaf5d938b5b3d
![](Pasted%20image%2020240108132715.png)
insert顺序写日志，日志异步同步到数据库中，由于事务的存在，缓存和日志的数据一定是一致的
#### Write Behind/Back方案
适合写多的，一致性没那么强的场景，如点赞
