---
title: 使用pac文件屏蔽那些不想访问的网站
date: 2018-06-05 11:13:41
tags: [pac,屏蔽网站]
---

## 概述

代理自动配置（Proxy auto-config，简称PAC），用于定义应用该如何自动选择适当的代理服务器来访问一个网址。一般使用在浏览器中，现在在一些便携式设备（Android设备，iOS设备）中，WIFI连接时也可以添加PAC协议来实现自动代理配置。

## 使用场景

在我们的场景中，我们有一些网站我们并不想去访问，譬如一些自动弹出的广告，某一个糟心的网站等等。这时就可以利用PAC文件来主动屏蔽这些网站。

## 使用方法

+ 编写PAC文件`filter.pac`
```PAC
// 我们不想访问的网站域名
var FILTERS = [
    "do_not_want_to_visit_1.com",
    "do_not_want_to_visit_2.com",
    "do_not_want_to_visit_3.com",
];

// 一个没有在使用的端口，因为我们不想访问这些网站
var PROXY = "PROXY 127.0.0.1:9999";
var DERECT = "DIRECT";

// PAC的主方法
function FindProxyForURL(url, host) {
    function rule_filter(domain) {
        for (var i = 0; i < FILTERS.length; i++) {
           if (domain.search(FILTERS[i]) !== -1) {
               return false;
           }
        }
        return true;
    }

    if (rule_filter(host) === true) {
        return DERECT;
    } else {
        return PROXY;
    }
}
```

+ 把PAC文件上传到网络上，且保证我们能直接访问到该文件。如果你有GitHub账号，你可以直接在你的项目中上传该文件。

+ 在浏览器插件（如`SwitchySharp`）中，或者是在WIFI链接时的代理设置->自动选项中，填入我们上传的地址（我们的场景是`https://github.com/oobspark/oobspark.github.io/blob/master/files/filter.pac`）。