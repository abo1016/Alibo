---
title: linux cron定时任务 环境变量坑
tags: [crontab]
id: '371'
categories:
    - linux
    - shell
date: 2019-01-12 17:06:53
---

##### 最近写了一个自动删除日志文件的shell脚本，在手动执行下一切正常。但是使用cron做定时任务时各种坑爹的问题。

脚本如下：

```shell
#!/bin/bash
#日志路径
cd /root/IOTC_Server/log/SSI_A146_0001_0003_2
#从15天前的时间往前删15天log
start_time=`date -d "-1 month" +%s`
for ((i=0;i<15;i++))
do
time=$((start_time + i*86400))
#删除ddhh日志
ymd=`date -d @$time +%Y%m%d`
d=`date -d @$time +%d`

echo "正在删除 $ymd 的日志" >> /root/del_iot_log.txt

ddhh_log=`ls|grep "$d"|grep "^$d"`
if [ -z "$ddhh_log" ];then
echo 'not find file'
else
rm -f $ddhh_log
fi

Device_log=`ls|grep "$d"|grep "^DeviceUsage$d"`
if [ -z "$Device_log" ];then
echo 'not find file'
else
rm -f $Device_log
fi

Connect_log=`ls|grep "$d"|grep "^ConnectResult$d"`
if [ -z "$Connect_log" ];then
echo 'not find file'
else
rm -f $Connect_log
fi

Relay_log=`ls|grep Relay$ymd`
if [ -z "$Relay_log" ];then
echo 'not find file'
else
rm -f $Relay_log
fi

sta_file=`ls|grep Statistics$ymd`
if [ -z "$sta_file" ];then
echo 'not find file'
else
rm -f $sta_file
fi

done
shell_time=`date "+%F %T"`
echo "$shell_time log删除完成!" >> /root/del_iot_log.txt
```

原来的被删除日志路径直接写的绝对路径，然后通过管道符找到要删除的文件名，最后通过`rm -f $log_path/文件名`命令删除。 手动执行后，正常删除查找到所有文件。但是通过cron执行此脚本，死活只能删除一个文件例如查找到123.txt 234.txt,执行时是`rm -f $log_path/123.txt 234.txt`,结果只有123.txt被删除掉 定位到应该是环境变量的问题，通过在脚本最前头引入环境变量，还有网上说的一些方法，结果还是一样。 最后还是通过`cd`先切换目标路径后 在进行脚本的删除执行。无问题 :astonished: 虽然解决了问题，一圈下来还是不知道是哪里出了问题，故此先做记录。