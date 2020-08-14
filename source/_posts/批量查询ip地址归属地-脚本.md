---
title: 批量查询IP地址归属地 脚本
tags: [批量ip, 脚本]
id: '345'
categories:
    - linux
    - shell
date: 2018-12-03 11:16:03
---

> 最近需要从服务器日志中获取请求ip的来源地，所以写了以下shell脚本

```shell
#!/bin/bash
count=`cat $1|wc -l`
n=1
cat $1 | while read m
do
ip=`echo $m | awk '{print $4}'`
addr=`curl -s https://ip.cn/index.php?ip=${ip}`
#淘宝这个接口有点问题，批量获取时会有请求不到数据的情况
#addr=`curl -s http://ip.taobao.com/service/getIpInfo.php?ip=${ip}|awk -F '"' '{print  $12,$20}'`
echo "`echo $m|awk '{print $1,$2,$3}'` ${addr} 总数：${count} 完成：${n}"
echo "`echo $m|awk '{print $1,$2,$3}'` ${addr}" >> ip_addr.txt
n=$(($n+1))
don
```