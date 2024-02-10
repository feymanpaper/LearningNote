### Go-zero
#### Goctl
生成model代码
```
goctl model mysql datasource --dir ./ --table fan  --url "root:351681578wdp@tcp(127.0.0.1:3306)/titok_relation"
```
生成rpc
```
goctl rpc protoc ./user.proto --go_out=. --go-grpc_out=. --zrpc_out=./
```
生成api
```
goctl api go --dir=./ --api applet.api
```
### 配置环境
### ETCD
### Redis
### MySQL
#### Consul
http://localhost:8500/
```
//启动
consul agent -dev
```
#### Prometheus
http://localhost:9090/

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
#### Grafana
http://localhost:3000/
```
sum(rate(http_server_requests_duration_ms_count{env="$env", service_group="$service_group", service_name="$service_name"}[1m])) by (path)
```
#### Jager
是用docker启动的，需要到官网找命令
http://localhost:16686/
```
docker run --rm --name jaeger \
  -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 4317:4317 \
  -p 4318:4318 \
  -p 14250:14250 \
  -p 14268:14268 \
  -p 14269:14269 \
  -p 9411:9411 \
  jaegertracing/all-in-one:1.53
```
### Kafka
```
## 创建Zookeeper容器
docker run -d --name zookeeper --network feyman -p 2181:2181 -t zookeeper
## 创建kafka容器
docker run -d --name kafka --network beyond -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 wurstmeister/kafka

cd /opt/kafka/bin

创建topic
./kafka-topics.sh --create --topic topic-like --bootstrap-server localhost:9092

查看topic信息
./kafka-topics.sh --describe --topic topic-like --bootstrap-server localhost:9092

生产消息
./kafka-console-producer.sh --topic topic-like --bootstrap-server localhost:9092

消费消息
./kafka-console-consumer.sh --topic topic-like --from-beginning --bootstrap-server localhost:9092
```

### 微服务
```
api
8888

rpc
user: 8080
count: 8081
relation: 8082
```
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
### Redis
查看某个key的大小
```
memory usage mykey
```
set a 1
大小为56B
set b 123a
大小为72B
#### 空间优化
https://www.freebuf.com/articles/database/367974.html
采用string存，100w数据需要50MB, 1亿需要5.6GB, 1个string占用56B
压测，redis 100w数据存string的情况下。开启100个goroutine，每个goroutine读请求1万次 key，100w并发读，耗时16-17s，QPS达到5.88-6.25w
压测，redis 100w数据存string的情况下。开启100个goroutine，每个goroutine写incr 1万次 key，100w并发写，耗时16-17s，QPS达到5.88-6.25w

采用hash的field val少于512时采用listpack存储的特性, 将100w数据分成3000个bucket，平均每个bucket存330个数据，每个bucket平均占用2.5KB，3000个桶大约占7MB。一亿数据占用700MB
相当于节省了85%的内存占用
压测，redis 100w数据存在hash分bucket的情况下。开启100个goroutine，每个goroutine请求1万次 key，100w并发读，耗时17-18s，QPS达到5.50-5.88w
压测，redis 100w数据存在hash分bucket的情况下。开启100个goroutine，每个goroutine写incr求1万次 key，100w并发写，耗时17-18s，QPS达到5.50-5.88w

读写性能为原来的94%，但空间却只为原来的12%

ps：如果把100w数据分到1000个bucket，平均每个bucket 1000个field value，此时hash底层不会用listpack而是用hash存储，每个bucket平均占用60KB，1000个桶占用60MB，一亿数据占用6G，空间占用太过庞大
分到100个bucket，平均每个bucket 10000个field，占用72KB，总共占用72MB，一亿数据7G
### 压测
grpc压测
ghz
```
ghz --insecure --proto=relation.proto --call=relation.Relation.FollowList -d '{
    "UserId": 1,
    "cursor": 0,
    "pageSize": 20,
    "sortType": 0,
    "ToId": -1
}' -c 100 -n 5000  127.0.0.1:8082
```

### 项目亮点
mysql设置合适的索引
识别热点操作：点赞，关注，评论; 查看热点用户信息，查看热点视频信息，粉丝列表, 评论列表
对于读多写少，如查看和修改用户信息，视频信息。采用旁路缓存+写数据库，删缓存来维持数据库和缓存一致性
singleflight保护数据库, 读如果redis存在直接返回，不存在则缓存空值，则查db，这个时候会singleflight防击穿。修改的话就先写数据库，删除缓存, 同时有singleflight防止删除大v导致击穿

对于读多写多，读如果redis存在直接返回，不存在则缓存空值，查db，singleflight防止击穿。对于修改，如新增加列表，新增计数，直接查看缓存是否存在，不存在就load上来，然后更新缓存，异步kafka更新数据库.

对于计数，不需要考虑消息顺序性，可以设置多个partition来提高吞吐量。kafka可以异步+批量更新
对于关注的状态, 有顺序性, 需要考虑顺序性.设置一个partition?

对于列表如何优化？
在redis上直接改, mysql深分页优化

如何快速识别是否关注，是否点赞？
bloom filter

jaeger链路追踪原理？etcd注册中心原理？

feed流如何设计?

对于用户信息中基本个人信息（用户名，id等等...）的计数信息（粉丝数，关注数，所有视频的获点赞数，所有视频的评论数，自身的喜欢数）。是否需要分开存呢？
如果全部放在user表，其他评论，点赞，关注的逻辑都要调用user rpc写，耦合性太强，而且user rpc承受的压力太大，因为user rpc的读请求也非常多

如果把计数分摊在各个服务，比如relation的计数（粉丝数，关注数）放在relation_rpc，以此类推，那么每次对user rpc的个人信息读，就需要请求好多次rpc（调用relation，like等等的微服务）。

因此用这种方案：user服务冗余这些计数信息，然后通过canal和消息队列来更新计数。

计数统计，为什么用string or hash而不用HyperLogLog，string or hash占用内存更低
`HyperLogLog`：存在一定误差，占用内存少，稳定占用 12k 左右内存，可以统计 2^64 元素
`HyperLogLog`主要是用于去重

单独分一个count表就不考虑了，需要改架构...

可以计算一下redis的结构，多少数据占用了多少内存, 100w用户的数据占用多少

TODO：加一个实时排行榜

查看视频的所有评论，按发布时间倒序,这种好去重，但是按照评论点赞数或者其他数字来排序，不好去重。可以采用
这样的话，看起来有两种处理方法：一种是摒弃掉实时刷新的机制、换成5分钟或者十分钟刷新一次 第二种是 由于点赞数会变，所以不
将其加入到排序规则中
调研一下抖音的功能可以发现：
前几个是热度（点赞数）最高的，这几个应该是没做分页直接拿的、后面几个按时间从新到日来展示（按时问做分 页由于时问不会变，所以完全没有重复或丢失的问题）
https://blog.csdn.net/qq_24664053/article/details/93197413

### 参考资料 
go-zero进行RPC调用、错误处理、JWT鉴权
https://juejin.cn/post/7272581426331254839#heading-10
一文教你搞定所有前端鉴权与后端鉴权方案，让你不再迷惘
https://juejin.cn/post/7129298214959710244?searchId=2024011511531288D4F42FCB2509035E49
微博数据库设计，提供了布隆过滤器查isFollow
https://www.zhihu.com/question/19715683/answer/598815840

听说你会架构设计？来，弄个微博系统
https://zhuanlan.zhihu.com/p/610915804

redis和mysql都可以批量查询，减少IO