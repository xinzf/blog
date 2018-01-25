## systemctl配置
```
# /usr/lib/systemd/system/wxrobot.service
[Unit]
Description=微信机器人服务
After=network.target

[Service]
EnvironmentFile=/etc/sysconfig/wxrobot
ExecStart=/usr/local/bin/wxrobot --conf /etc/wxrobot.json
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```
```
# /etc/sysconfig/wxrobot
MICRO_REGISTRY=consul
MICRO_REGISTRY_ADDRESS=192.168.2.104:8500
MICRO_SERVER_ADDRESS=127.0.0.1:37080
```

## supervisor配置
```
# /usr/local/etc/supervisor.d/wxrobot.ini
[program:wxrobot]
command=/usr/local/bin/wxrobot --conf /usr/local/etc/wxrobot.json
priority=2
redirect_stderr=true
autostart=false
stopsignal=INT
stdout_logfile=/usr/local/var/log/wxrobot.log
environment=MICRO_REGISTRY="consul",MICRO_REGISTRY_ADDRESS="192.168.2.104:8500",MICRO_SERVER_ADDRESS="127.0.0.1:37080"
```
