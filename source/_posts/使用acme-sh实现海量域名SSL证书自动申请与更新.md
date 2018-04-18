---
title: 使用acme.sh实现海量域名SSL证书自动申请与更新
date: 2018-04-18 16:56:40
tags:
---
## 需求
公司有个非常重要的业务：为摄影师们提供独立域名的个人网站建站服务。

考虑到安全性，及提升B格，我们给个人网站上了SSL手段。

从2017年起，各大证书商纷纷推出免费DV证书，让这个任务实现了零成本。

但是阿里云每个账号仅允许申请20个免费SSL证书，腾讯云再多也是50个。

当各个平台的羊毛薅完一遍之后，还是不够用，只能回到Let's Encrypt，频繁地在[sslforfree](https://www.sslforfree.com)上人肉申请证书。

该证书唯一的最大的不足，就是只有90天有效期，这就带来了后期非常繁琐的重复性工作。

如何通过工具自动化地完成这个流程，就是本文要解决的问题。

## 关于acme.sh
ACME全称The Automatic Certificate Management Environment，而[acme.sh](https://github.com/Neilpang/acme.sh)这个库，则能够在Linux上实现如下功能：

1. 自动向Let's Encrypt申请证书；
2. 自动调用各大云平台的api接口实现TXT解析配置；
3. 证书下发后自动部署到nginx；
4. 利用定时器，每60天自动更新证书，并完成自动部署。

## 流程
部署环境：Ubuntu 14.06 + nginx，域名注册与解析位于阿里云

### 安装acme.sh
``` bash
$ curl https://get.acme.sh | sh
```
或者
``` bash
$ wget -O -  https://get.acme.sh | sh
```
这个自动安装过程完成了以下几个步骤：
1. 拷贝sh脚本到`~/.acme.sh/`
2. 创建alias别名`acme.sh=~/.acme.sh/acme.sh`
3. 启动定时器

### 配置阿里云解析
运行如下命令，配置阿里云api接口的key和secret，其中的值需要到阿里云控制台中去寻找。
``` bash
$ export Ali_Key="sdfsdfsdfljlbjkljlkjsdfoiwje"
$ export Ali_Secret="jlsdflanljkljlfdsaklkjflsa"
```
这两个配置将永久保存在文件`~/.acme.sh/account.conf`中

### 为域名申请证书
运行如下命令，一键申请证书。
``` bash
$ acme.sh --issue --dns dns_ali -d www.example.com
```
证书申请成功后，保存在`~/.acme.sh/www.example.com`目录下

### 将证书部署到nginx
运行如下命令，自动将证书部署到nginx。
``` bash
$ acme.sh --install-cert -d www.example.com \
  --key-file       /path/to/keyfile/in/nginx/key.key  \
  --fullchain-file /path/to/fullchain/nginx/cert.pem \
  --reloadcmd     "nginx -s reload"
```
该命令中的参数将自动保存在`~/.acme.sh/www.example.com`目录下的`www.example.com.conf`文件里，定时器更新证书的时候实现自动部署。

### 配置nginx
在nginx的配置文件中配置如下：
```
server {
        listen 80;
        listen 443;
        server_name www.example.com;

        ssl on;
        ssl_certificate ./www.example.com.pem;
        ssl_certificate_key ./www.example.com.key;
        ssl_session_timeout 5m;
        ssl_session_cache shared:SSL:20m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers   on;
        location / {
            proxy_pass http://localhost:3000;
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        }
}
```

## 调用方式
在服务器上部署一个小nodejs服务，通过调用该服务的api接口，来运行对应的脚本，就能够实现新域名的证书申请+证书部署。

而证书的更新，就交给acme.sh，高枕无忧了~！
