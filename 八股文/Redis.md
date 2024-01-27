### 如何保证Redis和DB一致性
个人思考:
#### Cache Aside
适合读多写少的场景, 最终一致性
**先更新数据库，再删缓存**
保证两个操作都能执行成功
- 重试机制。
- 订阅 MySQL binlog，再操作缓存。
#### Write Through
适合读多写少的场景,强
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

### Redis为什么这么快
https://mp.weixin.qq.com/s/z4VjDaDDbspFz1rIBwazIA
![](Pasted%20image%2020240125215135.png)

### Redis事务和Lua脚本有什么区别？
https://pdai.tech/md/db/nosql-redis/db-redis-x-trans.html
#### Redis 的事务模式具备如下特点：
##### 保证隔离性；
我们可以将事务执行可以分为 EXEC 命令执行前和 EXEC 命令执行后两个阶段，分开讨论。EXEC 命令执行前在事务原理这一小节，我们发现在事务执行之前 ，Redis key 依然可以被修改。此时，可以使用 WATCH 机制来实现乐观锁的效果。

EXEC 命令执行后因为 Redis 是单线程执行操作命令， EXEC 命令执行后，Redis 会保证命令队列中的所有命令执行完 。 这样就可以保证事务的隔离性
##### 无法保证持久性；
数据库的持久性是指 ：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。
Redis 的数据是否持久化取决于 Redis 的持久化配置模式 。
没有配置 RDB 或者 AOF ，事务的持久性无法保证；
使用了 RDB模式，在一个事务执行后，下一次的 RDB 快照还未执行前，如果发生了实例宕机，事务的持久性同样无法保证；
使用了 AOF 模式；AOF 模式的三种配置选项 no 、everysec 都会存在数据丢失的情况 。always 可以保证事务的持久性，但因为性能太差，在生产环境一般不推荐使用。综上，redis 事务的持久性是无法保证的 。
##### 具备了一定的原子性，但不支持回滚；
命令入队时报错， 会放弃事务执行，保证原子性；命令入队时正常，执行 EXEC 命令后报错，不保证原子性；
##### 一致性的概念有分歧
**事务的一致性和预先定义的约束有关，保证了约束即保证了一致性**
假设在一致性的核心是约束的语意下，Redis 的事务可以保证一致性。

##### CAS操作实现乐观锁
实现incr
```
val = GET mykey
val = val + 1
SET mykey $val
```
有并发问题, 可以用watch
```
WATCH mykey
val = GET mykey
val = val + 1
MULTI
SET mykey $val
EXEC
```
##### Lua脚本
Lua脚本是redis事务的另外一种实现，但是比redis事务要更好
使用 Lua 脚本的好处 ：
- 减少网络开销。将多个请求通过脚本的形式一次发送，减少网络时延。
- 原子操作。Redis会将整个脚本作为一个整体执行，中间不会被其他命令插入。
- 复用。客户端发送的脚本会永久存在 Redis 中，其他客户端可以复用这一脚本而不需要使用代码完成相同的逻辑。

