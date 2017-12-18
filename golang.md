# Go编译器
建议使用版本`1.9.2`，可以使用`1.9.x`但不保证没有问题，`<1.9`的版本禁止使用

## 安装
很简单，去官网下载相应的系统版本，解压就可以直接使用了，这里用`centos 7`做一个示例
```
# 下载linux版
curl -LO "https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-amd64.tar.gz"
# 解压
tar zxvf go1.9.2.linux-amd64.tar.gz
# 移动到建议目录
mv go /usr/local
# 建立软链
ln -s /usr/local/go/bin/* /usr/local/bin
# 查看版本
go version
```
>为啥使用软链？因为`go`在寻找包时会用到`$GOROOT`，默认是`go`程序所在目录

## 配置
`$GOPATH`是非常重要的，建议指向用户目录
```
$GOPATH
    - bin #go编译生成的可执行程序存放位置
    - src #go源代码存放位置
```
> 没有配置`$GOPATH`默认使用`$HOME/go`也就是`~/go`目录

# 开发
为每一个项目编写详细的`README.md`是非常重要的，必须有。通过[GO语言文档](https://studygolang.com/pkgdoc)中文版快速查找了解内置方法使用。到[GO语言教程](http://www.runoob.com/go/go-tutorial.html)系统学习。

## 源代码目录结构
```
- src
    - pipifit #命名空间
        - project-1 #项目名称
            - example
                - api
                    - main.go
            - package-1 #一个包
                - http_test.go
            - package-2 #另一个包
        - hello
            - http
                - say.go
                - say_test.go
            - socket
                - say.go
                - say_test.go
            - glide.yaml
            - glide.lock
            - README.md
            - main.go
```
> project下面有main.go可以编译文件时，不建议有package文件

## 创建项目
必须使用[glide](https://github.com/Masterminds/glide)管理工具维护代码

### glide镜像
```yaml
#~/.glide/mirrors.yaml
repos:
- original: https://golang.org/x/crypto
  repo: https://github.com/golang/crypto
  vcs: git
- original: https://golang.org/x/image
  repo: https://github.com/golang/image
  vcs: git
- original: https://golang.org/x/mobile
  repo: https://github.com/golang/mobile
  vcs: git
- original: https://golang.org/x/net
  repo: https://github.com/golang/net
  vcs: git
- original: https://golang.org/x/sys
  repo: https://github.com/golang/sys
  vcs: git
- original: https://golang.org/x/text
  repo: https://github.com/golang/text
  vcs: git
- original: https://golang.org/x/tools
  repo: https://github.com/golang/tools
  vcs: git
```

### 无法下载package
国内对golang的官网通常会墙掉，导致无法下载`golang.org/x`下面的代码
```bash
glide mirror set https://golang.org/x/mobile https://github.com/golang/mobile --vcs git
glide mirror set https://golang.org/x/crypto https://github.com/golang/crypto --vcs git
glide mirror set https://golang.org/x/net https://github.com/golang/net --vcs git
glide mirror set https://golang.org/x/tools https://github.com/golang/tools --vcs git
glide mirror set https://golang.org/x/text https://github.com/golang/text --vcs git
glide mirror set https://golang.org/x/image https://github.com/golang/image --vcs git
glide mirror set https://golang.org/x/sys https://github.com/golang/sys --vcs git
```
### glide对依赖包路径解析问题
代码中会引入子包，glide分析出子包地址直接使用git下载
```
https://golang.org/x/net/context
```
但是git只允许下载全部包，所以下载包（install）时会报错

解决方法是，指定子包的全包地址
```yaml
package: https://golang.org/x/net/
subpackage:
 - context
```

## 代码编写
编写的代码必须用[golint](https://github.com/golang/lint)执行通过

## protobuf
有统一管理的地方

## 通用package
开发中如果使用到类似mysql、redis等通用功能时，必须使用下面提供的package
 - [微服务](https://github.com/micro/micro)
 - [WEB服务](https://github.com/astaxie/beego)
 - MySQL客户端
 - [redis客户端](https://github.com/go-redis/redis)
 - kafka客户端

## 运行参数
执行时加入或配置文件或环境变量

## 单元测试
编写单元测试是提高开发效率的有效手段之一

# 代码库管理
使用git库

## 分支
不允许在主干（master）分支进行开发，全部在其它分支开发然后合并

## 标签
实现一个比较大功能最好能打一个标签，便于发布和回滚

# 发布

## 版本号
每次发布必须要有版本号，如`1.2.345`

# 配套工具
可以根据个人喜好选择编写代码的工具，这里推荐`vim`版本必须`>8.0`，否则核心插件无法使用

## vim
使用vim编写go代码[vim-go](https://github.com/fatih/vim-go)是核心插件，依赖的工具包括[ctags](https://ctags.io/)、`guru`、[gocode](https://github.com/nsf/gocode)、[gotags](https://github.com/jstemmer/gotags)、[godep](https://github.com/tools/godep)、[gojson](https://github.com/ChimeraCoder/gojson)、[delve](https://github.com/derekparker/delve)。
>vim[基础环境配置](https://github.com/gaodongchen/vim)推荐。

## VS Code
额...
