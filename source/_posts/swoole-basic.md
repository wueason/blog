---
title: Swoole精华手记
date: 2020-07-14 10:22:42
tags: [Swoole]
---

### 知识点：

+ [可选回调](https://wiki.swoole.com/#/server/port?id=%e5%8f%af%e9%80%89%e5%9b%9e%e8%b0%83)

```
port 未调用 on 方法，设置回调函数的监听端口，默认使用主服务器的回调函数，port 可以通过 on 方法设置的回调有：

TCP 服务器
	onConnect
	onClose
	onReceive
UDP 服务器
	onPacket
	onReceive
HTTP 服务器
	onRequest
WebSocket 服务器
	onMessage
	onOpen
	onHandshake
```


+ [事件执行顺序](https://wiki.swoole.com/#/server/events?id=%e4%ba%8b%e4%bb%b6%e6%89%a7%e8%a1%8c%e9%a1%ba%e5%ba%8f)

```
所有事件回调均在 $server->start 后发生
服务器关闭程序终止时最后一次事件是 onShutdown
服务器启动成功后，onStart/onManagerStart/onWorkerStart 会在不同的进程内并发执行
onReceive/onConnect/onClose 在 Worker 进程中触发
Worker/Task 进程启动 / 结束时会分别调用一次 onWorkerStart/onWorkerStop
onTask 事件仅在 task 进程中发生
onFinish 事件仅在 worker 进程中发生
onStart/onManagerStart/onWorkerStart 3 个事件的执行顺序是不确定的
```