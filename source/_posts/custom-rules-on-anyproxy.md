---
title: AnyProxy的自定义规则
date: 2018-06-11 15:22:30
tags: [anyproxy]
---

## 概述

`AnyProxy`是一个开放式的HTTP代理服务器。

主要特性包括：

+ 基于`Node.js`，开放二次开发能力，允许自定义请求处理逻辑
+ 支持Https的解析
+ 提供GUI界面，用以观察请求



类似的软件还有`Fiddler`，`Charles`等。对于二次开发能力的支持，Fiddler 提供脚本自定义功能（`Fiddler Script`）。

`Fiddler Script`的本质其实是用`JScript.NET`语言写的一个脚本文件`CustomRules.js`，语法类似于C#，通过修改`CustomRules.js`可以很容易的修改http的请求和应答，不用中断程序，还可以针对不同的URI做特殊的处理。

但是如果想要进行更加深入的定制则有些捉襟见肘了，例如发起调用远程API接口等。当然如果你是C#使用者，这当然不在话下了。

我们都知道`Node.js`几乎可以做差不多任何事:)，而基于`Node.js`的`AnyProxy`则给予了二次定制更大的空间。

## 安装

因为是基于`Node.js`，故而Node支持的平台`AnyProxy`都能支持了。

`npm install -g anyproxy`

对于`Debian`或者`Ubuntu`系统，在安装`AnyProxy`之前，可能还需要安装 `nodejs-legacy`。

`sudo apt-get install nodejs-legacy`

## 启动

+ 命令行启动AnyProxy，默认端口号8001

`anyproxy`

+ 启动后将终端http代理服务器配置为127.0.0.1:8001即可

+ 访问http://127.0.0.1:8002 ，web界面上能看到所有的请求信息

## rule模块

`AnyProxy`提供了二次开发的能力，你可以用js编写自己的规则模块（rule），来自定义网络请求的处理逻辑。

![处理流程](images/anyproxy-process.png "处理流程")

例如我们想针对某些域名做检测，看经过`AnyProxy`代理的请求中是否包含了我们想要检测的那些域名。那么我们可以通过以下脚本实现：

+ 首先我们安装两个包

    `npm install redis`
    `npm install request`

+ 然后编写文件`check.js`

```javascript
// file: check.js
var redis   = require('redis')
var request = require('request')

var redisOn = true

var client = redis.createClient('6379', '127.0.0.1')

client.on("error", function(error) {
    console.log(error);
    var redisOn = false
})

var domainsListToCheck = [
    'domainToCheck1',
    'domainToCheck2',
    'domainToCheck3',
    'domainToCheck4',
    'domainToCheck5',
]

module.exports = {
  *beforeSendResponse(requestDetail, responseDetail) {

    var inList = false

    for (var i = 0; i < domainsListToCheck.length; i++) {

        inList = requestDetail.url.search(domainsListToCheck[i]) != -1
        if(inList){
            break
        }
    }

    if (inList) {

        var ua = requestDetail.requestOptions.headers['User-Agent'].toLowerCase()
        var ourAgent = ''

        if(ua.search('iphone') != -1){
            ourAgent = 'iphone'
        }

        if(ourAgent){
            
            if(redisOn){
                client.select('0', function(error){
                    client.set(ourAgent, '1', function(error, res) {
                        console.log(error, res)
                    })
                })
            }else{
                request({
                    url: 'https://keyvalue.immanuel.co/api/KeyVal/UpdateValue/lglm4ov9/'+ourAgent+'/1',
                    method: "POST",
                }, function(error, response, body) {
                    console.log(error, response, body)
                });
            }
        }
        return null
    }
  },
}
```

值得注意的是，我们在脚本中还是使用了一个本地Redis服务，如果你不想在本地启动一个Redis实例，你也可以使用`keyvalue.immanuel.co`。

`keyvalue.immanuel.co`是一个在线的Key-Value存储服务，完全免费。对于这种**临时的**，**不重要的**标记真是再方便不过了。个人使用下来觉得很赞。

## 使用自定义rule模块

`anyproxy --rule check.js`

## 了解更多

`AnyProxy`的更多功能可以参考[官方文档](http://anyproxy.io)。
