---
title: 如何在ubuntu上快速搭建shadowsocks
date: 2018-05-23 18:39:17
tags: ubuntu shadowsocks 
---
## 前期准备
购买海外vps，配置1核1G即可，操作系统选择Ubuntu。

海外vps推荐[bandwagon](https://bandwagonhost.com/aff.php?aff=32245)（俗称搬瓦工）和[vultr](https://www.vultr.com/?ref=7432142)。

通过nodejs的npm命令来安装shadowsocks是最快捷方便的方式，下面就来安装。

## 安装nodejs环境
执行命令更新系统包
``` bash
$ apt update
```

安装nodejs 8.x版本
``` bash
$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
$ apt-get install -y nodejs
```

## 安装pm2
pm2可以看做一个nodejs环境下的服务器，利用它来启动和维持shadowsocks服务
``` bash
$ npm -g i pm2
```

## 安装shadowsocks
执行命令安装shadowsocks
``` bash
$ npm -g i shadowsocks
```
然后，编辑配置文件：
``` bash
$ vim /usr/lib/node_modules/shadowsocks/config.json
```

修改后的文件内容如下：
``` js
{
    "server":"0.0.0.0",
    "server_port":8083,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":"myPassword",
    "timeout":600,
    "method":"aes-256-cfb"
}
```
从配置文件可以知道，对外的服务端口是`8083`，密码是`myPassword`，加密方式是`aes-256-cfb`

## 在pm2中启动shadowsocks服务
执行命令启动服务：
``` bash
$ pm2 start /usr/bin/ssserver
```

通过如下命令能够看到pm2下正常服务的应用
``` bash
$ pm2 ls
```

通过如下命令能够看到shadowsocks服务的日志
``` bash
$ pm2 logs ssserver
```

就此shadowsocks已经成功安装在服务器上了。

## 客户端安装
到github上去下载对应的客户端
 - windows客户端下载地址：https://github.com/shadowsocks/shadowsocks-windows/releases
 - MacOS客户端下载地址：https://github.com/SHAU-LOK/shadowsocks-mac-dmg/blob/master/ShadowsocksX-2.6.3.dmg
 - Android客户端下载地址：https://github.com/shadowsocks/shadowsocks-android/releases

（注：iPhone由于应用在App Store下架了，所以就自己去App Store找类似的shadowsocks客户端）


安装完毕之后，只需要配置服务器的ip地址、端口、密码、加密方式，就能正常使用了。
