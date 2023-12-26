https://mp.weixin.qq.com/s/PT1zpv3bvJiIJweN3mvX7g
map 中，key 的数据类型必须为可比较的类型，slice、map、func不可比较
### 并发冲突
map 不是并发安全的数据结构，倘若存在并发读写行为，会抛出 fatal error.
具体规则是：
（1）并发读没有问题；
（2）并发读写中的“写”是广义上的，包含写入、更新、删除等操作；
（3）读的时候发现其他 goroutine 在并发写，抛出 fatal error；
（4）写的时候发现其他 goroutine 在并发写，抛出 fatal error.
```
fatal("concurrent map read and map write")
fatal("concurrent map writes")
```
需要关注，此处并发读写会引发 fatal error，是一种比 panic 更严重的错误，无法使用 recover 操作捕获.

### 核心原理
渐进迁移
