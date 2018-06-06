---
title: 代理服务器的匿名级别
date: 2018-06-06 10:11:21
tags: [代理服务器,Proxy]
---

## 概述

代理服务器（Proxy Server）的基本行为就是接收客户端发送的请求后转发给其他服务器。代理不改变请求URI，会直接发送给前方持有资源的目标服务器。根据代理类型的不同，我们对于目标服务器的匿名程度也有所不同。

## 未使用代理

在没有经过代理服务器的情况下，目标服务端能获取到如下信息。

```
REMOTE_ADDR = your IP
HTTP_VIA = not determined
HTTP_X_FORWARDED_FOR = not determined
```

## 透明代理（`Transparent Proxy`） 

```
REMOTE_ADDR = proxy IP
HTTP_VIA = proxy IP
HTTP_X_FORWARDED_FOR = your IP
```
透明代理虽然可以直接“隐藏”你的IP地址，但还是可以从`HTTP_X_FORWARDED_FOR`查到你是IP地址。这也是我们一般所说的`Cache Proxy`。

## 匿名代理（`Anonymous Proxy`）

```
REMOTE_ADDR = proxy IP
HTTP_VIA = proxy IP
HTTP_X_FORWARDED_FOR = proxy IP
```
使用匿名代理，别人只能知道你用了代理，无法知道你是谁。这也是使用得比较广泛的一种代理方式。

## 混淆代理（`Distorting Proxy`）

```
REMOTE_ADDR = proxy IP
HTTP_VIA = proxy IP
HTTP_X_FORWARDED_FOR = random IP address
```
使用了混淆代理，别人还是能知道你在用代理，但是会得到一个假的IP地址。

## 高匿代理（`Elite proxy`或`High Anonymity Proxy`）

```
REMOTE_ADDR = Proxy IP
HTTP_VIA = not determined
HTTP_X_FORWARDED_FOR = not determined
```
使用高匿代理时，我们发现跟我们不使用代理是一样的，别人此时根本无法发现你是在用代理服务，这是最好的选择。
