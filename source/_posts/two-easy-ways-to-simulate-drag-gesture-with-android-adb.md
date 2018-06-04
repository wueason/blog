---
title: 使用安卓ADB工具模拟滑动操作的两种简易方式
date: 2018-06-04 09:11:35
tags: [adb,andriod,simulate,gesture,drag,swipe]
---

近日需要在安卓设备中模拟滑动操作，进行一番研究之后，发现了两种比较简易的方式。不能不感叹安卓`Android Debug Bridge (adb)`工具功能之强大。下面进入正文。

## 环境搭建

+ 接入设备并安装设备驱动（此过程请自行百度）

+ Windows系统中，从[官网下载](http://adbshell.com/downloads)`ADB Kits`并解压，譬如解压为`D:\adb`。

+ 为了方便起见，一般我们可以把`D:\adb`加入Windows环境变量中。此处我们直接由`cmd`进入adb目录

+ 执行`adb devices`，这时候我们就能看到之前接入的设备标识，这就说明adb已经可以正常使用了

![adb devices](images/adb-devices.jpg "adb devices")

## 第一种方式（`adb shell input swipe`)

`adb shell input swipe xStart yStart xEnd yEnd duration`

此方法比较简单，也是使用得比较广泛的一种，GOOGLE或BAIDU到的大量文章都是基于此方法实现的。

+ `adb shell input swipe 200 600 200 300 1000`，表示从坐标（200,600）这个点滑动到坐标（200,300），1000毫秒内完成。表现在屏幕上就是上滑过程。

## 第二种方式（`Monkey Script`)

`Drag(xStart, yStart, xEnd, yEnd, stepCount)`

`Money`命令是adb中用来测试程序稳定性的一个工具。根据参数的不同，它可以产生不同的测试效果。用它可以产生很多随机事件，当然，也可以使用`Monkey Script`来产生很多指定事件。`Monkey Script`，我们来了解一下。

+ Monkey也是属于adb工具的一部分，所以还是要先安装好adb，请参考文章第一部分

+ 编写`Monkey Script`脚本，并命名为`drag.mks`

    ```MonkeyScript
    count = 1
    speed = 1.0
    start data >> 
    Drag(200,600,200,300,100)
    UserWait(500)
    ```

+ `adb push drag.mks /data/local/`

+ `adb shell monkey -f /data/local/drag.mks 10`，将上滑操作执行10次