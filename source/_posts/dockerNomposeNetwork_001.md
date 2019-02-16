---
title: docker-compose network 入门踩坑
date: 2019-02-16 16:09:53
tags:
  - docker
  - docker-compose
---

docker-compose network 入门踩坑，窗口相互连接问题

<!-- more -->

首先，在 `docker-compose.yml` 文件尾部，全局 `networks` 部分定义了两个自定义网络，分别名为 `frontend`，`backend`。


```
networks:
    frontend:
    backend:
```

每个自定义网络都可以配置很多东西，包括网络所使用的驱动、网络地址范围等设置。但是，你可能会注意到这里 `frontend`、`backend` 后面是空的，这是指一切都使用默认，换句话说，在单机环境中，将意味着使用 `bridge` 驱动；而在 `Swarm` 环境中，使用 `overlay` 驱动，而且地址范围完全交给 Docker 引擎决定。

然后，在前面`service`s中，每个服务下面的也有一个 `networks` 部分，这部分是用于定义这个服务要连接到哪些网络上。

```
services:
    nginx:
        ...
        networks:
            - frontend
    php-fpm:
        ...
        ports:
            - "9000"
        networks:
            - frontend
            - backend
    mysql:
        ...
        networks:
            - backend
```

在这个例子中，
- nginx 接到了名为 frontend 的前端网络；
- mysql 接到了名为 backend 的后端网络；
- 而作为中间的 php 同时连接了 frontend 和 backend 网络上。

连接到同一个网络的容器，可以进行互连；而不同网络的容器则会被隔离。 所以在这个例子中，nginx 可以和 php-fpm 服务进行互连，php 也可以和 mysql 服务互连，因为它们连接到了同一个网络中； 而 nginx 和 mysql 并不处于同一网络，所以二者无法通讯，这起到了隔离的作用。

> 处于同一网络的容器，可以使用服务名访问对方。比如，`./site/index.php` 里，可以使用的 `mysql` 这个服务名去连接的数据库服务器,`nginx`通过 'php-fpm:9000'访问 'php-fpm'

使用 `nginx` 中连接 `php`
```
server {
	...
	location ~ \.php$ {
		...
		fastcgi_pass php-fpm:9000;
	}
}

```
`fastcgi_pass php-fpm:9000` 中 `php-fpm` 表示 `php-fpm`这个服务