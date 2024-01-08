### 如何保证Redis和DB一致性

读多写少: 
https://www.bilibili.com/video/BV1Z14y1D74Y?p=18&vd_source=108d23f95683578313bdaf5d938b5b3d
数据库+缓存双写+冷热分离(过期时间+自动expireTime过期)+load db&write cache+缓存惊群解决方案(随机过期时间)+缓存穿透解决方案(db load null->cache empty data)+数据库&缓存一致性方案(写和读, 分布式锁+读double check)+分布式锁串行转并发方案(读操作的分布式锁, tryLock, 自动超时)