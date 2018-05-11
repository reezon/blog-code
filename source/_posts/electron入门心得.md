---
title: electron入门心得
date: 2018-04-25 19:25:50
tags: electron
---
## 安装
#### 在macOS上全局安装electron
``` bash
$ npm i -g electron --unsafe-perm=true --allow-root
```

#### 部署quick-start
``` bash
$ git clone https://github.com/electron/electron-quick-start
```

## 辅助工具
### electron-forge
#### 1.安装electron-forge
``` bash
$ npm install -g electron-forge
```

#### 2.初始化项目
``` bash
$ electron-forge init my-new-project --lintstyle=standard --template=react
```

#### 3.启动项目
``` bash
$ electron-forge start
```

#### 4.打包项目
``` bash
$ electron-forge package
```

### electron-vue
``` bash
# 安装 vue-cli，使用脚手架样板代码初始化项目
$ npm install -g vue-cli
$ vue init simulatedgreg/electron-vue my-project
```

## 主进程与渲染进程
electron的开发主要涉及到两个进程的操作：Main主进程，Renderer渲染进程。

主进程主要通过nodejs、chromium、native apis来实现一些系统或底层的操作，比如操作剪贴板等。

渲染进程主要通过chromium来实现web界面。

两个进程通过`ipcMain`和`ipcRenderer`来进行通信

#### 主进程向渲染进程发消息
main.js文件：
``` js
// 当页面加载完成时，会触发`did-finish-load`事件。
win.webContents.on('did-finish-load', () => {
  win.webContents.send('main-process-messages', 'webContents event "did-finish-load" called');
});
```

renderer.js文件
``` js
// 引入ipcRenderer对象
const electron = require('electron');
const ipcRenderer = electron.ipcRenderer;
// 设置监听
ipcRenderer.on('main-process-messages', (event, message) => {
  console.log('message from Main Process: ' , message);  // Prints Main Process Message.
});
```

#### 渲染进程向主进程发消息
renderer.js文件
``` js
// 引入ipcRenderer对象
const electron = require('electron');
const ipcRenderer = electron.ipcRenderer;
ipcRenderer.send('asynchronous-message', 'hello');

ipcRenderer.on('asynchronous-reply', (event, arg) => {
  console.log('asynchronous-reply: %O %O', event, arg);
});
```

main.js文件
``` js
ipcMain.on('asynchronous-message', (event, arg) => {
  // 返回消息
  event.sender.send('asynchronous-reply', 'ok');
});
```
