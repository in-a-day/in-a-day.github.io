---
title: redis安装
tags:
  - redis
categories: redis
date: 2021-04-11 11:44:00
---

本文基于centos7

```bash
# 添加EPEL仓库
sudo yum install epel-release
# 安装redis
sudo yum install redis
# 启动redis
sudo systemctl start redis
# 开启自启
sudo systemctl enable redis
# 重启redis
sudo systemctl restart redis
```
默认redis配置文件位于`/et/redis.conf`

