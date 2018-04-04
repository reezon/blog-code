---
title: Docker实战
date: 2018-04-03 22:21:14
tags: Docker
---
## 需求
为用户实现独立域名绑定个人网站，但是一台阿里云ECS最多只允许5个域名进行备案，因此需要部署n台服务器，以满足5n个用户的个人网站。

这n台服务器的server代码和nginx配置都是相同的，最适合使用Docker技术来实现。

## 步骤
### 1.在Ubuntu中安装Docker
``` bash
$ apt-get update && apt-get install docker.io
```

### 2.使用pull命令从官方镜像库中拽取一个 Ubuntu 14.04的image
``` bash
$ docker pull ubuntu:14.04
```
### 3.如果拽取速度太慢，可以安装阿里云的Docker加速器
``` bash
$ mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://03gzvkg6.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```
阿里云的镜像地址是：dev.aliyun.com/search.html

### 4.运行image
``` bash
$ docker run -it --rm ubuntu:14.04 bash 
```
其中：--rm表示退出后就销毁该container

### 5.编写自己的Dockerfile
新建一个目录，在目录下运行
``` bash
$ touch Dockerfile
```
然后编辑这个Dockerfile文件
``` bash
FROM ubuntu:16.04
MAINTAINER picbling <it@picbling.com>

# 安装环境
RUN apt-get update && apt-get install -y \
    git \
    nginx \
    htop \
    vim \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 安装node
RUN curl -sL https://deb.nodesource.com/setup_6.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# 拉取server项目代码
RUN cd /root \
    && git clone https://xxx/server.git \
    && cd server \
    && npm i \
    && npm run build

# 拉取nginx配置
RUN cd /root && \
    git clone https://xxx/nginx.git && \
    cd nginx && \
    cp nginx.conf /etc/nginx/  && \
    cp -rf sites-enabled /etc/nginx/

# 启动pm2
RUN npm i -g pm2

# 运行server
CMD ["pm2", "start", "NODE_ENV=production /root/server/server.js"]

EXPOSE 22
EXPOSE 80
```

### 6.在Dockerfile目录下运行命令来编译
``` bash
$ docker build -t picbling/homepage-server-ubuntu-16.04 .
```

### 7.将image上传到云端
``` bash
$ docker login
$ docker push picbling/homepage-server-ubuntu-16.04
```
