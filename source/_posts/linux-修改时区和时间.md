---
title: linux 修改时区和时间
tags: [修改时区时间]
id: '300'
categories:
    - linux
date: 2018-09-06 09:19:08
---

**查看时区** `date -R` **设置时区** `tzselect` **复制相应的时区文件，替换系统时区文件；或者创建链接文件** `cp /usr/share/zoneinfo/$主时区/$次时区 /etc/localtime` 例如：上海 `cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime` **将当前时间和日期写入BIOS，避免重启后失效** `hwclock -w`