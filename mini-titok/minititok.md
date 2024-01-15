
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

### 参考资料 
go-zero进行RPC调用、错误处理、JWT鉴权
https://juejin.cn/post/7272581426331254839#heading-10
一文教你搞定所有前端鉴权与后端鉴权方案，让你不再迷惘
https://juejin.cn/post/7129298214959710244?searchId=2024011511531288D4F42FCB2509035E49