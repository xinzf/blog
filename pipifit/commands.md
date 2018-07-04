# curl
```bash
# 用户token生成
curl -v service.intranet.pipifit.com/ucenter/token -d 'uid=32'

# 语意分析接口
curl "$WXROBOT_HTTP_SERVER/rpc" -H "Content-Type: application/json" \
-d '{"service":"qa.question","method":"Question.Intent","params":[{"Text":"苹果晚上能吃吗？"}]}'
```

# ssh
```bash
# 建立线上通道
ssh -NCf  -p 2231 gaodongchen@39.107.67.111 \
-L30218:192.168.30.218:22 -L30808:192.168.80.8:22 -L30217:192.168.30.217:22 \
-L60195:192.168.60.195:22 -L10112:192.168.10.112:22
# micro
ssh gaodongchen@127.0.0.1 -p 10112
# api
ssh gaodongchen@127.0.0.1 -p 30217
# wap
ssh gaodongchen@127.0.0.1 -p 30218
# consul
ssh gaodongchen@127.0.0.1 -p 60195
# admin
ssh gaodongchen@127.0.0.1 -p 30808

# mysql
ssh -NCf -L33061:192.168.10.116:3306 -p 10112 gaodongchen@127.0.0.1
# redis
ssh -NCf -L63791:192.168.10.113:6379 -p 10112 gaodongchen@127.0.0.1

# old jump
ssh -NCf gaodongchen@123.56.17.248 \
-L10125:192.168.1.25:22 -L12151:192.168.2.151:22 \
-L10107:192.168.1.7:22 -L12150:192.168.2.150:22 \
-L10118:192.168.1.18:22 -L10122:192.168.1.22:22 \
-L10110:192.168.1.10:22 -L12153:192.168.2.153:22 \
-L10117:192.168.1.17:22 -L10119:192.168.1.19:22 -L10120:192.168.1.20:22 \
-L10102:192.168.1.2:22 -L10121:192.168.1.21:22 -L12152:192.168.2.152:22

# frontend
ssh root@127.0.0.1 -p 10125
ssh root@127.0.0.1 -p 12151
# backend
ssh root@127.0.0.1 -p 10107 X
ssh root@127.0.0.1 -p 12150
# kafka
ssh root@127.0.0.1 -p 10117
ssh root@127.0.0.1 -p 10119
ssh root@127.0.0.1 -p 10120
# im
ssh root@127.0.0.1 -p 10110
ssh root@127.0.0.1 -p 12153
# es
ssh root@127.0.0.1 -p 10118
ssh root@127.0.0.1 -p 10122
# web
ssh root@127.0.0.1 -p 10102
# push
ssh root@127.0.0.1 -p 12152
# log
ssh root@127.0.0.1 -p 10121

# 旧数据库
ssh -NCf -L 37060:192.168.1.13:3306 -L 37061:192.168.1.29:3306 -L 37062:192.168.1.15:3306 \
gaodongchen@123.56.17.248

# 104
ssh gaodongchen@39.107.97.26 -p 37616

# ngrok
ssh gaodongchen@39.107.97.26

# vps
ssh root@69.171.68.132 -p 26718
```

# scp
```bash
scp -P 30218 /tmp/html.zip gaodongchen@127.0.0.1:/tmp
scp ~/Downloads/html.zip gaodongchen@192.168.2.104:/tmp
scp -P 30218 www@127.0.0.1:/tmp/abc.zip /tmp/
```

# tcpdump
```bash
tcpdump -nnA -c 5 -s 0 -i enp2s0 '(tcp[(tcp[12]>>2):4] = 0x47455420) or (tcp[(tcp[12]>>2):4] = 0x48545450) and host 192.168.2.244'

tcpdump -nnA -c 5 -s 0 -i enp2s0 '(tcp[32:4] = 0x47455420) or (tcp[32:4] = 0x48545450) and host 192.168.2.244'

tcpdump -nnA -c 5 -s 0 -i enp2s0 'tcp port !22 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0) and host 192.168.2.244'

tcpdump -nnX -c 5 -s 0 -i enp2s0 'tcp port !22 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0) and host 192.168.2.244'
```

# mysql
```bash
# rm-2ze1qf6q5w8w86m8brw.mysql.rds.aliyuncs.com
mysql -h127.0.0.1 -P33061 -uucenter_writer -pE3KyHk7NCkE2j2
mysql -h127.0.0.1 -P33061 -uplan_writer -pnhayG6cg28F2fL
mysql -h127.0.0.1 -P33061 -ucms_writer -ppUmf4a7rg8FZ4P
mysql -h127.0.0.1 -P33061 -uspread_writer -pHb8WhdfN93Z6TY

mysql -h127.0.0.1 -P37060 -upipi_core_writer -pFx8UKriHK4k
mysql -h127.0.0.1 -P37061 -ucard_writer -pFx8UKriHK4k
mysql -h127.0.0.1 -P37062 -umall_writer -pFx8UKriHK4k
```

```sql
-- 导出为分班学员列表
SELECT `user`.id as 'user id', `user`.`nickname` as 'nickname',`profile`.`real_name` as `realname`,
`user`.`mobile` as 'mobile',
`course`.`order_sn` as 'order_sn', FROM_UNIXTIME(`course`.`create_date`) as 'create time'
FROM `course_user` as `course`
LEFT JOIN `user` as `user` on `user`.`id` = `course`.`uid`
LEFT JOIN `user_profile` as `profile` on `user`.`id` = `profile`.`uid`
WHERE `course`.`status` = 0  
into outfile '/tmp/course.csv' fields terminated by ','optionally enclosed by ''lines terminated by '\n';
```

# redis
```bash
# r-2ze3e8bc8e625df4.redis.rds.aliyuncs.com
redis-cli -h 127.0.0.1 -p 63791 -a eK4DB862hfPtVr
```

```redis
config set protected-mode "no"
```

# micro
```
# 创建机器人
micro query pipi.micro.wechat.robot Robot.Create '{}'
# 机器人详情
micro query pipi.micro.wechat.robot Robot.Profile '{"TaskId":"911c84ff-b5a5-486b-9f3e-e6116c9aa2f7"}'
# 启动机器人
micro query pipi.micro.wechat.robot Robot.Start '{"TaskId": "23a08367-ab47-4097-84b6-318b517f06d5"}'

# 消息发送
micro query pipi.micro.wechat.robot Robot.Send '{\
"TaskId": "bdc49d66-d82a-4c95-89f6-4c864868a82e",\
"SendMessage": "少华是个大帅哥",\
"SendTo":"@8aad055467f67c63b33357c95c27bc5c",\
"SendType": "text"}'

```

# ss-local
```
# vps
ss-local -s 69.171.68.132 -p 443 -l 1080 -m aes-256-cfb -k U2FsdGVkX18jtxG+KjbG1TrZc7YWzY82j64T71LQcmE=
```
# awk
```
# base syntax
awk '{pattern + action}' {filenames}

# Poe 33794712
echo "I am Poe,my qq is 33794712"|awk -F '[ ,]+' '{print $3" "$7}'

# user count is  62
awk '{count++;print $0;} END{print "user count is ",count}' /etc/passwd
```

# mount
```
mount /dev/mapper/xo-home /volumes/home
```
