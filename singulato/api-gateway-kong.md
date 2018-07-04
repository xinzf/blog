# Virtual Box
The command is set intnet in Virtual Box.
```cmd
VBoxManage.exe dhcpserver add --netname intnet --ip 192.168.1.1 --netmask 255.255.255.0 --lowerip 192.168.1.100 --upperip 192.168.1.200 --enable
```
# Kong
## Installation
Create a Docker network
```bash
$ docker network create kong-net
```
Start your database
```bash
$ docker run -d --name kong-database \
    --network=kong-net \
    -p 9042:9042 \
    cassandra:3
```

Prepare your database
```bash
$ docker run --rm \
    --network=kong-net \
    -e "KONG_DATABASE=cassandra" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    kong:latest kong migrations up
```
Start Kong
```bash
$ docker run -d --name kong \
    --network=kong-net \
    -e "KONG_DATABASE=postgres" \
    -e "KONG_PG_HOST=kong-database" \
    -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" \
    -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
    -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
    -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
    -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" \
    -p 8000:8000 \
    -p 8443:8443 \
    -p 8001:8001 \
    -p 8444:8444 \
    kong:latest
```
allow set `KONG_CUSTOM_PLUGINS=auth4toke`

## Plugin
```bash
$ luarocks pack kong-plugin-auth4token 0.0.1-0
```
# Apply
Enable Plugin
```bash
$ curl -i -X POST \
    --url http://localhost:8001/plugins/ \
    --data 'name=auth4token' \
    --data 'config.url=http://app.pre.singulato.com/user/center/v1/login/user/token'
```
Create a api
```bash
$ curl -i -X POST \
    --url 'http://localhost:8001/apis/' \
    --data 'name=mall_car_app_cards' \
    --data 'hosts=apis.pre.singulato.com' \
    --data 'uris=/mall/car/app/cards' \
    --data 'methods=GET' \
    --data 'upstream_url=http://app.pre.singulato.com/sku/center/v1/app/cards'
```
# Bugs

## Kong
 - Kong is not support that set lua_code_cache.

# Reference
 * [API gateway 之 kong 基本操作 （三）](https://www.cnblogs.com/chenjinxi/p/8724564.html)
 * [token验证代码示例](https://github.com/FlyingShit-XinHuang/kong-auth-demo)
 * [lua路径](https://blog.csdn.net/rheostat/article/details/17284493)
 * [API gateway 之 kong 安装 （二）](http://www.cnblogs.com/chenjinxi/p/8723558.html)
 * [lua中的http与https请求](https://blog.csdn.net/xiejunna/article/details/53445342)
 * [API Gateway——KONG插件开发（官网译文）](https://www.jianshu.com/p/548586579ff2)
