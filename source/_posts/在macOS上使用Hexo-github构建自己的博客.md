---
title: 在macOS上使用Hexo+github构建自己的博客
date: 2018-03-30 18:29:16
tags: hexo github
---
### 在macOS上安装Hexo
首先需要安装[nodejs](https://nodejs.org/en/)环境

然后运行
``` bash
$ npm i -g hexo
```

### 初始化blog项目
新建一个目录，然后进入目录，运行
``` bash
$ hexo init
```

### 在github上创建博客项目
在github上创建一个如下名字的项目
> xxx.gihub.io

### 配置blog项目的发布方式
安装git发布插件
``` bash
$ npm install hexo-deployer-git --save
```

打开文件 **_config.xml** , 编辑如下内容

```
deploy:
  type: git
  repo: https://github.com/YourgithubName/YourgithubName.github.io.git
  branch: master
```

### 本地运行
运行如下命令
``` bash
$ hexo clean
$ hexo generate
$ hexo server

```

### 部署到github
运行如下命令
``` bash
$ hexo clean
$ hexo generate
$ hexo deploy

```
就能自动发布到xxx.github.io了
