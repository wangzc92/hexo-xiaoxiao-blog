title: 涨姿势：你真的会用console调试代码吗？
author: wangzc
tags:
  - js
  - console
categories:
  - 前端基础
  - Javascript
  - ''
date: 2017-03-08 13:22:00
---
>如果你是开发测试人员，一定对console不陌生。如果是初学者，恐怕只知道console.log()啦。对于整个的console家族，你又了解多少呢？我们使用console对js进行调试，但js原生中默认是没有console对象的，它是宿主对象，也就是浏览器提供的内置对象，所以浏览器不同，console对象也会有些许差异。使用console.log(console)打印出浏览器中的console对象，看看你用过哪些？

下面是chrome浏览器截图：

![console](http://ofi3qxlvd.bkt.clouddn.com/images/point/console0.jpg)

仔细看一下，还真的不少呢。接下来介绍几个常用的console方法。
## 用来打印消息
* 打印字符串：console.log()
* 打印提示消息：console.info()
* 打印警告消息：console.warn()
* 打印错误消息：console.error()
* 打印调试信息：console.debug()

这几个方法都可以接受多个参数，逗号分隔。它会在每次输出结尾添加换行符。没有返回值就返回undefined。另外这五种方法都可以使用printf风格占位符，字符（%s）、整数（%d或%i）、浮点数（%f）和对象（%o）四种。如果第一个参数是格式字符串，方法会将依次用后面的参数替换占位符，然后再进行输出。五种方法示例如下：

![console.log](http://ofi3qxlvd.bkt.clouddn.com/images/point/console1.jpg)

## 统计次数console.count()
有时候我们需要统计一个函数被调用了几次，一般都是加入一个变量然后输出。现在，我们可以直接使用console.count()来帮助我们统计并输出。

![console.count](http://ofi3qxlvd.bkt.clouddn.com/images/point/console2.jpg)

## 判断真假console.assert()
我们写代码的时候总会碰到判断某个变量是否为真，这个时候可以console.assert()来判断，当表达式为false的时候，输出信息到控制台。

![console.assert](http://ofi3qxlvd.bkt.clouddn.com/images/point/console3.jpg)

## 查看对象信息console.dir()
如果想要看详细的对象信息，我们可以使用 console.dir，将一个 JavaScript 对象的所有属性和属性值显示成一个可交互的列表，它还能打印出函数等。

![console.dir](http://ofi3qxlvd.bkt.clouddn.com/images/point/console4.jpg)

## 打印成表格console.table()
可以将传入的对象或数组这些复合数据以表格的形式输出。

![console.table](http://ofi3qxlvd.bkt.clouddn.com/images/point/console5.jpg)

## 输出信息分组console.group()和console.groupEnd()
如果输出信息比较多，可以进行分组管理。

![console.group](http://ofi3qxlvd.bkt.clouddn.com/images/point/console6.jpg)

> console方法还有很多，这里就不一一列举了，想了解更多请自行查阅官方API文档。掌握这些方法，对我们调试代码很有帮助。

> 原文链接：http://www.wangzc.cc/points-console/