# https

## nginx
```
    server {
        listen       8443 ssl http2 default_server;
        listen       [::]:8443 ssl http2 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        ssl_certificate "/home/gaodongchen/.nginx/etc/pki/server.crt";
        ssl_certificate_key "/home/gaodongchen/.nginx/etc/pki/private/server.key";
        ssl_session_cache shared:SSL:1m;
        ssl_session_timeout  10m;
        ssl_ciphers HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers on;

        # Load configuration files for the default server block.
        # include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

}
```

# TLS/SSL

## CA生成
如果要在浏览器显示出安全的绿锁标志，自己颁发的证书肯定不好使，得花钱向第三方权威证书颁发机构申请购买。
### 第三方CA生成流程
先生成CSR证书请求 ---->  然后自己保留私钥KEY ---> 然后将 CSR请求发给第三方CA机构 比如赛门铁克  ---> 等待CA颁发证书---》部署到服务器\
三方的流程我们一般是：   商务沟通 --->代理或CA公司发来合同-->合同确认OK  --》下单---> 确认订单 --> 财务付款成功 -->CA邮件和电话双向确认域名所有权--->....其他确认....--->颁发证书
### 自己生成
```bash
# 生成key
openssl genrsa -des3 -out ca.key 2048
# 生成CA
openssl req -new -x509 -nodes -subj "/CN=gaodc.com" -days 365 -key ca.key -out ca.crt
```

## CRT生成
```bash
# 生成key
openssl genrsa -des3 -out server.key 2048
# 生成CSR
openssl req -new -key server.key -out server.csr
# 生成CRT
openssl x509 -req -in server.csr -CA ca.pem -CAkey ca.key -CAcreateserial -days 365 -out server.crt
```

# Redis

关闭保护模式
```
config set protected-mode "no"
```

# SSH使用笔记

## 开机启动sshd服务
```
chkconfig --list
chkconfig --level 35 sshd on
```
## 权限
```
.ssh 700
.ssh/authorized_keys 600
.ssh/config 600
```
**创建OpenSSH密钥**
```
ssh-keygen -C user@host.domain
```
**代理**
```
ssh -NCf -i ~/.ssh/gdc.pem -D 8484 ubuntu@54.169.148.222
ssh -NCf -i c:\Users\phper\.ssh\yanjiao.pem -D 8484 ubuntu@52.74.189.143
ssh -NCf -D 8484 53f42d165973cacf850002e5@kaixinfan-gaodongchen.rhcloud.com
```

**通道**
```
screen ssh -L40001:192.168.8.88:22 root@ssh.domain.com
ssh -NCf -L40001:192.168.8.88:22 root@ssh.domain.com
```
## 配置
编辑`~/.ssh/config`

```
Host application.localhost
	User username
	Port 22
	IdentityFile ~/.ssh/rhcloud_rsa
	IdentitiesOnly yes
```
# PuTTY

## Plink的使用技巧

```
plink -P 29127 -N -C -D 8080 root@vps.gaodc.com
plink -N -C -L 3306:192.168.10.2:3306 xiangzhi@123.56.25.152
plink -N -C -L 22101:192.168.30.4:22 -L 22102:192.168.30.5:22 -L 22103:192.168.30.6:22 xiangzhi@123.56.25.152
```

# EMQ

emqttd | eMQTT

 * cluster 中的一个node意外退出，并不会影响client的连接和发送。server会自动将这个client链接到其它node上。
 * 在reids进行修改，不需要重新启动emqttd服务就会生效。
 * 当kafka意外出现状况，会导致emqttd服务崩溃
 * emqtt_plugin_redis 中的密码一定是plain
 * 开发测试打开reloader
 * 修改插件的配置后需要重启插件
 * 关闭retain*配置，防止登录收到最后一条消息

## 需要打开的插件

```
./bin/emqttd_ctl plugins load emqttd_reloader
./bin/emqttd_ctl plugins load emqttd_plugin_redis
./bin/emqttd_ctl plugins load emqttd_plugin_blackhole
```

## rebar.config

```
{erl_opt, [debug_info, report]}.
{cover_enabled, true}.
{cover_print_enabled, true}.
{clean_files, ["ebin/*.beam"]}.
{deps,[
    {ekaf, ".*", {git, "https://github.com/helpshift/ekaf.git"}}
]}.
{sub_dirs, ["src"]}.
```
 
## redis中添加用户

```
hset mqtt_user:root password 123456
hset mqtt_user:root is_superuser 1
sadd mqtt_acl:root "pubsub /World"
sadd mqtt_subs:root "/World"
```

# kafka

 - broker的启动需要系统有足够的swap。
 - kafka做集群，一定要设置host/address，否则访问不了。
 - zookeeper集群需要 echo 1 > /tmp/zookeeper/myid， 否则启动失败

## Quick Start
```
bin/zookeeper-server-start.sh config/zookeeper.properties
bin/kafka-server-start.sh config/server.properties

bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
bin/kafka-topics.sh --list --zookeeper localhost:2181

bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test --from-beginning
```

## python扩展
```
pip install kafka-python
```

# ELS

## 安装elasticsearch
```
curl -LO https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.3/elasticsearch-2.3.3.tar.gz
tar zxvf elasticsearch-2.3.3.tar.gz
useradd -r elastic
mkdir -p /var/elasticsearch/data
mkdir -p /var/log/elasticsearch/logs
chown elastic.elastic -R /var/elasticsearch/
chown elastic.elastic -R /var/log/elasticsearch/
chown elastic.elastic -R /opt/elasticsearch-2.3.3/
```

### 启动
```
su - elastic -c '/opt/elasticsearch-2.3.3/bin/elasticsearch -d'
```

### 测试命令
```
curl -XGET '192.168.30.6:9200/sample/_search?pretty&q=*'
```


## 安装logstash

```
$ rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
$ cat|tee /etc/yum.repos.d/logstash.repo <<EOF
[logstash-2.3]
name=Logstash repository for 2.3.x packages
baseurl=https://packages.elastic.co/logstash/2.3/centos
gpgcheck=1
gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
EOF
```

```
yum install java logstash
```

### 启动

```
/opt/logstash -f /etc/logstash/conf.d/sample.conf
```

### 微信日志监控配置
```
input {
    file {
        path => "/var/log/nginx/weixin/*.log"
        start_position => beginning
        codec => "json"
    }
}

filter {

}

output {
    elasticsearch {
        hosts => ["192.168.30.6"]
        index => "weixin"
        document_type => "logs"
        codec => "json"
    }
}
```

### nginx日志监控配置
```
input {
    file {
        path => "/var/log/nginx/payment/*.log"
        start_position => beginning
        codec => "json"
    }
}

filter {

}

output {
    elasticsearch {
        hosts => ["192.168.30.6"]
        index => "monitor"
        document_type => "logs"
        codec => "json"
    }
}
```

### PHP日志监控配置

```
input {
    file {
        path => "/var/log/weixin/payment/*.log"
        start_position => beginning
        codec => "json"
    }
}

output {
    elasticsearch {
        hosts => ["192.168.30.6"]
        index => "monitor"
        document_type => "logs"
        codec => "json"
    }
}
```

sample
```
input {
    file {
        path => "/tmp/sample.log"
        start_position => beginning
        codec => "json"
    }
}
output {
    elasticsearch {
        # default "logstash-%{+YYYY.MM.dd}"
        index => "sample"
        # default "plain"
        codec => "json"
    }
    stdout {
    }
}
```

# CentOS

## 查看命令日期格式设置
- 临时更改显示样式，当回话结束后恢复原来的样式
	export TIME_STYLE='+%Y-%m-%d %H:%M:%S'
- 永久改变显示样式，更改后的效果会保存下来
	修改/etc/profile文件，在文件内容末尾加入
	export TIME_STYLE='+%Y-%m-%d %H:%M:%S'
	执行如下命令，使你修改后的/etc/profile文件配置内容生效
	source /etc/profile 
## 忽略大小写（按tab）
bind 'set completion-ignore-case on'
## MD5
echo -n '1024'|md5sum
## 更改PS
PS1="[\033[1;31;49m\u\033[1;35;49m@\033[1;35;49m\h \033[1;34;49m\W\033[0m]\$"
export PS1

## 临时添加swap交换分区（重启后失效）

```
dd if=/dev/xvda of=/home/swap bs=1024 count=512000
mkswap /home/swap
swapon /home/swap
```

# erlang

## 编译安装
```
rebar get-deps compile

erl -noshell -pa /usr/lib/erlang/lib/ekaf-1.5.4/deps/gproc/ebin -pa /usr/lib/erlang/lib/ekaf-1.5.4/deps/kafkamocker/ebin -s test_ekaf test -s init stop
```

## beam加载
加载过一次的beam，如果重新编译，不重启erlang服务是不会生效的

## record
在shell 模式下rd 代表record
```
1> rd(u1,{a,b,c}).
u1
2> rd(u2,{b,c}).
u2
3> Users=[#u1{a=12,b="ligaoren",c=true},#u1{a=23,b="zen",c=false}].
[#u1{a = 12,b = "ligaoren",c = true},
 #u1{a = 23,b = "zen",c = false}]
4> [begin #u1{b=B,c=C}=U, #u2{b=B,c=C} end|| U<-Users].
[#u2{b = "ligaoren",c = true},#u2{b = "zen",c = false}]
5>  
```
## 函数

### 代码
**清除代码**
code:purge(pipi_collector).

**加载代码**
code:load_file(pipi_collector).

### 进程
**系统中所有进程**
processes().

# MongoDB

默认端口 27017

## 在centos中启动命令
```
mkdir /var/mongod
mkdir -p /var/run/mongod
mkdir -p /var/mongod/data
./bin/mongod --logpath /var/mongod/mongod.log --pidfilepath /var/run/mongod/mongod.pid --dbpath /var/mongod/data > /var/run/mongod/server.log 2>&1 &
```

# MySQL

## 初始root密码设置
```
The server creates a 'root'@'localhost' superuser account. The server's action with respect to a password for this account depends on how you invoke it:

With --initialize but not --initialize-insecure, the server generates a random password, marks it as expired, and writes a message displaying the password:

[Warning] A temporary password is generated for root@localhost:
iTag*AfrH5ej

With --initialize-insecure, (either with or without --initialize because --initialize-insecure implies --initialize), the server does not generate a password or mark it expired, and writes a warning message:

[Warning] root@localhost is created with an empty password ! Please
consider switching off the --initialize-insecure option.
```

# Git

## 基础命令
查看全局配置
```
git config --global -l
```
用户信息设置
```
git config --global user.name "username"
git config --global user.email "username@localhost"
```

显示颜色
```
git config --global color.ui true
git config --global color.diff true
git config --global color.status true
git config --global color.branch true
```
别名设置
```
git config --global alias.st status
git config --global alias.ci commit
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.dc dcommit
git config --global alias.rb rebase 
git config --global alias.revert checkout
```
分支查看
```
git branch -a
git branch -r
git branch -v 查看各个分支最后一次提交
```
删除分支
```
git branch -d branch-1
git branch -D branch-1
git branch -d -r origin/branch-1 
```
分支合并
```
git merge remote-origin/branch-1
git branch --no-merged
git branch --merged
```
分支切换
```
git checkout branch-1
git checkout -b branch-1
git checkout -b branch-1 remote-origin/branch-1
```

检出远程分支
```
git fetch remote-origin
```

## 配置示例
```
[core]
    repositoryformatversion = 0 
    filemode = true
    bare = false
    logallrefupdates = true
    editor = gvim
[remote "origin"]
    url = git@192.168.57.86:cms.git
    fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
    remote = "origin"
    merge = refs/heads/master
[i18n]
	commitEncoding = UTF-8
	logOutputEncoding = gbk
```

## 服务器搭建
添加git帐号
```
sudo groupadd -r git
sudo useradd -g git -s /usr/bin/git-shell git
```
创建空白仓库
```
mkdir project.git
cd project.git
git --bare init
```

## 常用配置
```
#.gitconfig
[user]
	email = gaodongchen@gmail.com
	name = gaodongchen
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
	editor = vim
    excludesfile = ~/.gitignore
[diff]
	tool = vimdiff
[difftool]
	prompt = false
[alias]
    df = difftool
    st = status
    ci = commit
    co = checkout
    br = branch
[color]
    diff = auto
    status = auto
    branch = auto
    interactive = auto
[push]
default = simple
```

## 忽略配置
```
#.ignorefile

*sw*
*.log
*.tmp
tags

.eunit
deps
*.o
*.beam
*.plt
erl_crash.dump
ebin
rel/example_project
.concrete/DEV_MODE
.rebar
```

# Tmux

## 常用配置
```
#.tmux.conf
unbind C-b

set -g prefix C-b
set -g default-shell /bin/zsh
setw -g mode-keys vi

bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
```

# Vim

diff shell
```bash
#!/bin/sh
#/usr/local/bin/diff
# 指定vimdiff的路径.
DIFF="/usr/bin/vimdiff"
# svn提供第六和第七个参数作为base和本地最新的文本作为输入
LEFT=${6}
RIGHT=${7}
#调用vimdiff做比较
$DIFF $LEFT $RIGHT
```
变更subversion配置
```
#~/.subversion/config
diff-cmd = /usr/local/bin/diff
```
