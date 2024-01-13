Prometheus 监控服务, 由于默认的9090端口被clashx占用，因此使用9010端口作为UI
```
prometheus --web.listen-address=:9010 &
```
查看端口
```bash
lsof -i:9090
```