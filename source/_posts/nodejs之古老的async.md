---
title: nodejs之古老的async
date: 2018-04-11 14:09:29
tags: nodejs
---
在 Promise和async/await出现之前，nodejs只能使用async库来解决“回调金字塔问题”

把async中一些常用的语法记录下来，以免遇到上古项目的时候需要

## 安装
``` bash
$ npm install async --save
```

## 主要方法
### series(task, [callback])
多个函数依次执行，之间没有数据交换
``` js
async.series([
    function(callback){
      console.log(1)
      callback(err, 1)
    }, function(callback){
      console.log(2)
      callback(err, 2)
    }, function(
      console.log(3)
      callback(err, 3)
    }],function(err, results){
    // results = [v1, v2, v3]
})
```
详细解释：
1. 依次执行一个函数数组中的每个函数，每一个函数执行完成之后才能执行下一个函数。
2. 如果任何一个函数向它的回调函数中传了一个error，则后面的函数都不会被执行，并且将会立刻将该error以及已经执行了的函数的结果，传给series中最后的那个callback。
3. 将所有的函数执行完后(没有出错),则会把每个函数传给其回调函数的结果合并为一个数组，传给series最后的那个callback。
4. 还可以以json的形式提供tasks。每一个属性都会被当作函数来执行，并且结果也会以json形式传给series中最后的那个callback。这种方式可读性更高

### waterfall(tasks, [callback])

多个函数依次执行，且前一个的输出为后一个的输入
``` js
async.waterfall([
    function(callback){
      console.log(1)
      callback(null, 1)
    }, function(n, callback){
      console.log(2)
      callback(null, 2)
    }, function(n, callback){
      console.log(3)
      callback(null, 3)
    }], function(err, results){
    // results = 3
})
```
详细解释：
1. 按顺序依次执行多个函数，每一个函数产生的值，都将传给下一个函数，如果中途出错，后面的函数将不会执行，错误信息以及之前产生的结果，都传给waterfall最终的callback
2. 该函数不支持json格式的tasks

### parallel(tasks,[callback])
多个函数并行执行
``` js
async.parallel([
    function(callback){
      console.log(1)
      callback(null, 1)
    }, function(callback){
      console.log(2)
      callback(null, 2)
    }, function(callback){
      console.log(3)
      callback(null, 3)
    }],function(err, results){
    // results = [1, 2, 3]
})
```
详细解释：
1. 并行执行多个函数，每个函数都是立刻执行，不需要等待其他函数先执行。传给最终callback的数组中的数据按照tasks声明的顺序，而不是执行完成的顺序
2. 如果某个函数出错，则立刻将err和已经执行完的函数的结果值传给parallel最终的callback。其它为执行完的函数的值不会传到最终数据，但要占个位置。
3. 同时支持json形式的tasks，其最终callback的结果也为json形式

### whilst(test, fn, callback)
循环执行任务
``` js
var count = 0
async.whilst(
    function(){
      return count < 3
    },
    function(cb){
        console.log(count)
        count++
    },
    function(err){
    }
)
```
详细解释：
1. 第一个函数用于控制循环次数
2. 在循环中，异步调用时产生的值实际上被丢弃了，因为最后的callback只能传入错误信息
3. 第二个函数fn需要接受一个函数的cb, 这个cb最终必需被执行，用于表示出错或正常结束

## Demo
详细代码Demo可见[async_demo](https://github.com/alsotang/async_demo)
