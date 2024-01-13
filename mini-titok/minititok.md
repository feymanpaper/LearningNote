
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



Consul:
http://localhost:8500/
Prometheus:
http://localhost:9010/
Grafana:
http://localhost:3000/
