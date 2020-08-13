---
title: linux cpu占用率监控脚本
tags: []
id: '290'
categories:
  - - all-blog
    - linux
    - shell
date: 2018-08-29 17:51:30
---

**/proc/stat：** 这个文件包含了所有CPU活动的信息，该文件中的所有值都是从系统启动开始累计到当前时刻。可以利用其中信息计算cpu的利用率。 `cat /proc/stat` ![PXK2tO.png](https://s1.ax1x.com/2018/08/29/PXK2tO.png) 每行每个参数的意思为（以第一行为例，单位：jiffies，1jiffies=0.01秒）： user（1062726189）：从系统启动开始累计到当前时刻，用户态的CPU时间，不包含 nice值为负进程。 nice（0）：从系统启动开始累计到当前时刻。 system（264108894）：从系统启动开始累计到当前时刻，nice值为负的进程所占用的CPU时间。 idle（2695159767）：从系统启动开始累计到当前时刻，除硬盘IO等待时间以外其它等待时间。 iowait（2950374）：从系统启动开始累计到当前时刻，硬盘IO等待时间。 irq（0）：从系统启动开始累计到当前时刻，硬中断时间。 softirq（45899110）：从系统启动开始累计到当前时刻，软中断时间。 CPU时间=user+nice+system+idle+iowait+irq+softirq。 CPU利用率=(idle2-idle1)/(cpu2-cpu1)\*100。

```shell
#!/bin/bash

interval=3
cpu_num=`cat /proc/stat | grep cpu[0-9] -c`

start_idle=()
start_total=()
cpu_rate=()
while (true) 
do
    start=$(cat /proc/stat | grep "cpu " | awk '{print $2" "$3" "$4" "$5" "$6" "$7" "$8}')
    start_idle[${cpu_num}]=$(echo ${start} | awk '{print $4}')
    start_total[${cpu_num}]=$(echo ${start} | awk '{printf "%.f",$1+$2+$3+$4+$5+$6+$7}')
    sleep ${interval}

    end=$(cat /proc/stat | grep "cpu " | awk '{print $2" "$3" "$4" "$5" "$6" "$7" "$8}')
    end_idle=$(echo ${end} | awk '{print $4}')
    end_total=$(echo ${end} | awk '{printf "%.f",$1+$2+$3+$4+$5+$6+$7}')
    idle=`expr ${end_idle} - ${start_idle[$cpu_num]}`
    total=`expr ${end_total} - ${start_total[$cpu_num]}`
    idle_normal=`expr ${idle} \* 100`
    cpu_usage=`expr ${idle_normal} / ${total}`
    cpu_rate[${cpu_num}]=`expr 100 - ${cpu_usage}`
    echo "The Average CPU Rate : ${cpu_rate[${cpu_num}]}%"

    echo "------------------"

done
```