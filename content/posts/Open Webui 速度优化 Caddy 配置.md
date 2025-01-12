---
title: "Open Webui 速度优化 Caddy 配置"
date: 2025-01-12T15:42:54+08:00
draft: false
---

加速 Open Webui 访问速度，主要是让客户端缓存静态文件和 /api/models。
配置如下
```
everse_proxy ip:port
 header /_app/immutable/* Cache-Control "max-age=2592000"
 header /api/models Cache-Control "max-age=2592000"
```
就可以实现加速访问，模型改变多的把下面的删了，没删的话要更改模型就打开 /api/models 清除缓存就好了