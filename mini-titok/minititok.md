
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
### 缓存和DB一致性
SingleFlight, 布隆过滤器
热点数据放在redis里，读如果redis存在直接返回，不存在则缓存空值，则查db，这个时候会有布隆过滤器和singlefight防击穿
写，如果redis存在直接写，如果redis不存在则从db reload，然后在redis写，然后用kafka异步同步到mysql里

对于用户信息这种热点数据，读多写少, 则用kafka异步, 一个partition或者保证同步mysql
对于点赞，关注这种热点操作，读多写多, 则用kafka分partition+批量同步mysql

### 参考资料 
go-zero进行RPC调用、错误处理、JWT鉴权
https://juejin.cn/post/7272581426331254839#heading-10
一文教你搞定所有前端鉴权与后端鉴权方案，让你不再迷惘
https://juejin.cn/post/7129298214959710244?searchId=2024011511531288D4F42FCB2509035E49