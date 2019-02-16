---
title: Docker Restart 策略
date: 2019-02-16 15:09:53
tags:
---
### Restart策略

使用`-–restart`参数可以指定一个restart策略,来指示在退出时容器应该如何重启或不应该重启。
```
docker run --restart=always ...
```

当容器启用restart策略时，将会在docker ps显示Up或者Restarting状态。也可以使用docker events命令来生效中的restart策略。

### Docker支持如下restart策略：

#### `no`
容器退出不自动重启,默认值

#### `on-failure[:max-retries]`
只在容器以非0状态码退出时重启,可配置重启次数

#### `unless-stopped`
始终重启容器，不过当daemon启动是，容器已经是停止状态，不要尝试启动它

#### `always`
始终重启容器，docker daemon 会一直尝试重启容器，直至成功，生产环境建议配置该值

### 重启延迟
容器的每次重启尝试延迟是上一次的双倍，通常是 100ms,200ms, 400ms, 800ms,... 直到超过 `on-failure` 或者 docker 退出，或者容器被删除，重启成功后让会重置延迟

查看重启次数
```
$ docker inspect -f "{{ .RestartCount }}" container-name
```

或者获取上一次容器重启时间：
```
$ docker inspect -f "{{ .State.StartedAt }}" container-name
```

