Prometheus 监控服务, 由于默认的9090端口被clashx占用，使用brew services start会失败
启动prometheus
```
cd /opt/homebrew/etc
prometheus --config.file=prometheus.yml
```

```bash
# 查看端口被哪个进程占用
lsof -i:9090
# 查看进程占用了哪个端口
lsof -nP -p 76440 | grep LISTEN
# 查看进程号
ps -e | grep prometheus
```

Consul
http://localhost:8500/
Prometheus
http://localhost:9090/
Grafana
http://localhost:3000/
Jager
是用docker启动的，需要到官网找命令
http://localhost:16686/

```
sum(rate(http_server_requests_duration_ms_count{env="$env", service_group="$service_group", service_name="$service_name"}[1m])) by (path)
```

### 微服务
划分为
用户微服务，计数微服务，关系微服务
好处:
把计数统一放在计数微服务，好处是查用户信息时只需要查一次微服务可以得到各种计数信息,否则要查多个微服务

坏处:
关注的时候要操作计数和关系微服务，原来只需要操作一个

如何保证点赞的实时性

### redis和DB考虑
获取个人信息，在redis+singleflight，且计数数据也在user服务的情况下，qps可以达到5000

SingleFlight, 布隆过滤器
热点数据放在redis里，读如果redis存在直接返回，不存在则缓存空值，则查db，这个时候会有布隆过滤器和singlefight防击穿
写，如果redis存在直接写，如果redis不存在则从db reload，然后在redis写，然后用kafka异步同步到mysql里

对于用户信息这种热点数据，读多写少, 则用kafka异步, 一个partition或者保证消息顺序性同步mysql
对于点赞，关注这种热点操作，读多写多, 则用kafka分partition+批量同步mysql

relation操作:
关系操作表:
id userid followerid deletedat createdat
是否有必要建立二级索引？可以用userid作为主键？

redis的存储为:
userid, followerset
这个时候就存在大key了，如何解决大key带来的性能抖动？

关系计数表: 
粉丝
专门用一个count表来存储
id, countkey, countval
方便横向扩展

redis存储为:
countkey:countval

关注一个人，需要增加这个人的粉丝数和粉丝id, 如果是同时写库还能用事务，如果两者都用kafka异步写入如果失败了怎么办？

如果user微服务，不冗余计数的话，查询一个人的粉丝列表，需要先到followservice, 然后查到id到userservice, 再到countservice，链路有三层，是否有链路过长的问题

如果user微服务不冗余计数字段的话，如果计数微服务挂了，用户刷视频的时候就拿不到这些计数数据了，可以在user表冗余这些计数字段，每隔一段时间把异步把计数字段同步到user冗余的字段里？？？

### 参考资料 
go-zero进行RPC调用、错误处理、JWT鉴权
https://juejin.cn/post/7272581426331254839#heading-10
一文教你搞定所有前端鉴权与后端鉴权方案，让你不再迷惘
https://juejin.cn/post/7129298214959710244?searchId=2024011511531288D4F42FCB2509035E49