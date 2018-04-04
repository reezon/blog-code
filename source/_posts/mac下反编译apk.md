---
title: mac下反编译apk
date: 2018-04-04 14:09:29
tags:
---
## 工具
- apktool：用于反编译apk文件
- dex2jar：用于将反编译出的class.dex转换成classes-dex2jar.jar
- jd-gui：用于阅读classes-dex2jar.jar源码
- signAPK：给重新打包后的apk签名

### 安装apktool
安装地址：
https://ibotpeaches.github.io/Apktool/install/

根据教程将apktool安装好即可。

注意：需要先安装java sdk。
下载地址：http://www.oracle.com/technetwork/java/javase/downloads/jdk9-downloads-3848520.html

### 安装dex2jar
下载并解压dex2jar-2.0
为dex2jar-2.0添加权限
``` bash
$ chmod -R 777 dex2jar-2.0
$ chmod +x d2j_invoke.sh
```

### 安装jd-gui
使用brew安装
``` base
$ brew cask install jd-gui
```
注意：只能运行在jdk 1.8版本下。否则按照此文档对jdk版本进行更改：http://blog.csdn.net/YoungStunner/article/details/78699864

## 反编译流程
### 使用apktool反编译apk包
运行如下命令来反编译：
``` bash
$ apktool d xxx.apk
```
用压缩软件unzip achiever打开StabilityTest.apk，然后解压出位于根目录下的classes.dex

### 使用dex2jar将classes.dex转换为jar包
运行如下命令来转换：
``` bash
$ sh d2j-dex2jar.sh classes.dex
```

### 使用jd-gui打开classes-dex2jar.jar
如果觉得jd-gui查看代码不方便，还可以通过File->Save All Sources导出一个classes-dex2jar.src.zip，将classes-dex2jar.src.zip解压以后，导入到Sublime阅读代码。

### 修改完smali源码后，使用apktool重新打包生成apk
运行如下命令来打包：
``` bash
$ apktool b xxx.apk
```

### 使用sign.jar进行签名
运行如下命令来签名：
``` bash
$ java -jar sign.jar xxxx.apk
```
