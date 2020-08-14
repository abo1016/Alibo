---
title: linux的一些常用命令
tags: [linux, 命令]
id: '35'
categories:
    - linux
date: 2018-05-11 11:43:14
---

格式：`pstree -p` 以树状图显示进程，还显示进程PID。 格式：`pstree <pid>` 格式：`pstree -p <pid>` 以树状图显示进程PID为的进程以及子孙进程，如果有-p参数则同时显示每个进程的PID。 ![](/wp-content/uploads/2018/05/20180511112155_32168.png) `netstat -tunpl` 列出所有tcp和udp连接 ![](/wp-content/uploads/2018/05/20180511113613_30081.png)

> telnet 命令

格式： `telnet ip prot` 测试域名解析：

```
root@vagrant-ubuntu-trusty-64:~# telnet ipc.iloveismarthome.com
Trying 47.92.39.114…
```

测试socket端口：

```
root@vagrant-ubuntu-trusty-64:~# telnet ipc.iloveismarthome.com 8***
Trying 47.92.39.114…
Connected to ipc.iloveismarthome.com.
Escape character is ‘^]’.
{“welcome”: “welcome to huabei3 IPC Server”}
```