---
layout: post
title:  "electron-ssr 在 Ubuntu18.04 无法使用的解决方案"
categories: Linux
tags: Linux Ubuntu electron-ssr
excerpt: Ubuntu 下 electron-ssr 无法使用解决方法
---


# Ubuntu 下 electron-ssr 无法使用解决方法

昨天安装上 Ubuntu, 装完常用软件后, 发现 electron-ssr 无法使用, 软件配置没有问题, 查看日志文件也没发现什么有价值的信息.

搜了一圈, 发现其实是 python 没设置好. Ubuntu 默认是有安装 python3 的, 但是终端内输入 python 会提示没有安装. 原因是没有链接命令.

```bash
$ sudo in -s /usr/bin/python3 /usr/bin/python
```

执行完, 重启下 electron-ssr 就行了~!