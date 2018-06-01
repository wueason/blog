---
title: 作为正向代理的Apache Traffic Server
date: 2018-05-31 15:14:35
tags: [Apache Traffic Server,正向代理,ForwardProxy]
---

## 对比

![Comparing HTTP Intermediaries](images/comparing_http_intermediaries.png "Comparing HTTP Intermediaries")

## 安装

+ 环境：`ubuntu 16.04`

+ `sudo apt-get install trafficserver`

## 配置

更改`records.config`配置，一般为`/etc/trafficserver/records.config`

```config
CONFIG proxy.config.url_remap.remap_required INT 0
CONFIG proxy.config.http.cache.http INT 1
CONFIG proxy.config.reverse_proxy.enabled INT 0                    
```

如果不想返回头中包含有ATS信息，如
```config
Proxy-Connection: keep-alive
Server: ATS/3.2.4
```

可以参照以下配置

```config
CONFIG proxy.config.http.response_server_enabled INT 0
CONFIG proxy.config.http.insert_age_in_response INT 0
CONFIG proxy.config.http.insert_request_via_str INT 0                 
CONFIG proxy.config.http.anonymize_insert_client_ip INT 0               
CONFIG proxy.config.http.insert_squid_x_forwarded_for INT 0           
CONFIG proxy.config.reverse_proxy.enabled INT 0                         
CONFIG proxy.config.url_remap.remap_required INT 0                      
```

## 启动

`sudo service trafficserver restart`

## 使用正向代理

配置代理服务器：`IP:8080`，8080为默认端口

