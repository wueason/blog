---
title: LNMP技术栈在Docker中的使用
date: 2019-01-20 10:37:16
tags: ['docker', 'lnmp']
---

### 目标

LNMP技术栈是Web开发中流行的技术栈之一，本文的目标是，利用docker搭建一套LNMP服务。

好，废话不多说，我们直入主题。

### Docker的安装

Docker CE（Community Edition）社区版本本身支持多种平台的安装，如Linux，MacOS，Windows等操作系统，此外，还支持AWS，Azure等云计算平台。

如果你使用的是Windows 10，那么你可以直接[`Docker Desktop for Windows`](https://hub.docker.com/editions/community/docker-ce-desktop-windows)。要使用此工具，你需要开启你Windows中的`Hyper-V`服务和`BIOS`中的`Virtualization`选项。

笔者使用的是Windows 7操作系统，直接使用[Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/)，下载并安装即可。

![docker-toolbox](images/docker-toolbox.png "docker-toolbox")

### 使用到的镜像

本文中会使用到以下三个基础镜像：

+ nginx:1.15
+ php:7.1-fpm
+ mysql:5.7

三个镜像都是官方提供的镜像，官方镜像保证了稳定性的同时，同时也保留了一些扩展性，使用起来比较方便。

我们先把三个镜像下载到本地备用。打开`Docker Quickstart Terminal`，并执行：

```bash
docker pull nginx:1.15
docker pull php:7.1-fpm
docker pull mariadb:10.3
```

### 常规方法

首先我们使用`docker`的基本命令来创建我们的容器。

+ MariaDB

打开`Docker Quickstart Terminal`后，执行：

```
cd lnmp
docker run --name mysql -p 3306:3306 \
    -v $PWD/mysql:/var/lib/mysql -d mariadb:10.3
```

查看服务状态：

```
mysql -h192.168.99.100 -uroot -p123123 -e "status"
```

此处返回服务器状态信息

![mariadb-status](images/mariadb-status.png "mariadb-status")

+ PHP-FPM

```
docker run --name php-fpm --link mysql:mysql -p 9000:9000 \
    -v $PWD/html:/var/www/html:ro -d php:7.1-fpm
```
```
    --name php-fpm：
        自定义容器名

    --link mysql:mysql
        与mysql容器关联，并将mysql容器的域名指定为mysql

    -v $PWD/www:/var/www/html:ro
        `$PWD/www`是宿主机的php文件目录
        `/var/www/html`是容器内php文件目录
        `ro`表示只读。
```
官方docker中已经包含的PHP的部分基本扩展，但是很显然这并不能满足大多数的使用场景。

因此，官方还提供了`docker-php-ext-configure`，`docker-php-ext-install`和
`docker-php-ext-enable`等脚本供我们使用，可以更方便的安装我们的扩展。

此外，容器还提供对`pecl`命令的支持。

我们基于此安装我们常用一些扩展。

```
docker-php-ext-install pdo pdo_mysql
pecl install redis-4.0.1 && \
    pecl install xdebug-2.6.0 \
    docker-php-ext-enable redis xdebug
```

当然我们也可以选择直接编译安装。

```
curl -fsSL 'http://pecl.php.net/get/redis-4.2.0.tgz' \
    && tar zxvf redis-4.2.0.tgz \
    && rm redis-4.2.0.tgz \
    && ( \
        cd redis-4.2.0 \
        && phpize \
        && ./configure \
        && make -j "$(nproc)" \
        && make install \
    ) \
    && rm -r redis-4.2.0 \
    && docker-php-ext-enable redis
```

+ Nginx

```
docker run --name nginx -p 80:80 --link php-fpm:php \
    -v $PWD/default_host.conf:/etc/nginx/conf.d/default.conf:ro \
    -v $PWD/html:/usr/share/nginx/html:ro \
    -d nginx:1.15
```

    --name nginx：
       自定义容器名

    --link php-fpm:php
       与php-fpm容器关联，并将php-fpm容器的域名指定为php

    -v $PWD/default_host.conf:/etc/nginx/conf.d/default.conf:ro
       替换host文件

    -v $PWD/html:/usr/share/nginx/html:ro \
       替换网站根目录

+ 总结

至此，我们依次启动了mysql，php-fpm和nginx容器（顺序很重要，因为他们有依赖关系）。打开浏览器，访问`http://192.168.99.100/`，就是见证奇迹的时刻。

### 高阶

以上是比较常规的一种方式，也稍显麻烦。下面介绍`docker-composer`的配置方式。

```
version: '3'
services:
    mysql:
        image: mariadb:10.3
        volumes:
            - mysql-data:/var/lib/mysql
        environment:
            TZ: 'Asia/Shanghai'
            MYSQL_ROOT_PASSWORD: 123123
        command: ['mysqld', '--character-set-server=utf8']
        ports:
            - "3306:3306"
        networks:
            - backend
    php:
        image: "mylnmp/php:v1.0"
        build:
            context: .
            dockerfile: Dockerfile-php
        ports:
            - "9000:9000"
        networks:
            - frontend
            - backend
        depends_on:
            - mysql
    nginx:
        image: "mylnmp/nginx:v1.0"
        build:
            context: .
            dockerfile: Dockerfile-nginx
        ports:
            - "80:80"
        networks:
            - frontend
        depends_on:
            - php
volumes:
    mysql-data:

networks:
    frontend:
    backend:
```

具体可参考我的GitHub项目[`lnmp-container`](https://github.com/wueason/lnmp-container)
