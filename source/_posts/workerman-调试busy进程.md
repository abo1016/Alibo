---
title: workerman 调试busy进程
tags: [workerman, 调试]
id: '288'
categories:
    - PHP
date: 2018-08-29 17:18:11
---

*   使用`php start.php status`查看wokerman状态，进程状态为busy，明对应进程正在处理业务，正常情况下业务处理完毕对应进程会恢复为idle状态，这一般情况下不会有什么问题。若进程持续很久都没有恢复到idle状态，这说明进程业务已经出现异常。

#### 利用strace+lsof命令定位

1.  利用`php start.php status`命令找出busy状态的进程pid。

[![workerman](http://doc.workerman.net/images/d1903ed65ef2f3b0850e84ccbedc52aa.png "workerman")](http://doc.workerman.net/images/d1903ed65ef2f3b0850e84ccbedc52aa.png "workerman") 图中busy的进程的pid为11725和11748

2.  `strace` 跟踪进程 挑选一个进程pid(这里选择11725)，运行`strace -ttp 11725` 显示如下

[![](http://doc.workerman.net/images/7ce9f36da926f670949609dcdc593ab4.png)](http://doc.workerman.net/images/7ce9f36da926f670949609dcdc593ab4.png) 可以看到进程在不断的循环poll(\[{fd=16, events=....的系统调用，这是在等待fd为16的描述符可读事件，也就是在等这个描述符返回数据。 如果没有显示任何系统调用，保留当前终端，重新再打开一个终端，运行kill -SIGALRM 11725(给进程发送一个闹钟信号)，然后看strace的终端是否有响应，是否阻塞在某个系统调用上。如果仍然没有显示任何系统调用说明程序很可能处于业务死循环中，参考页面下部引起进程长时间busy的其它原因 第2项 解决。 如果系统阻塞在epoll\_wait或者select系统调用是正常情况，这说明进程已经处于idle状态。

3.  lsof 查看进程描述符 运行lsof -nPp 11725 显示如下

[![](http://doc.workerman.net/images/27bd629c3a1ac93f9f4b535d01df2ac1.png)](http://doc.workerman.net/images/27bd629c3a1ac93f9f4b535d01df2ac1.png) 述符16对应的是16u的记录(最后一行)，能看fd=16的描述符是一个tcp连接，远程地址是101.37.136.135:80，说明进程应该是在访问一个http资源，循环poll(\[{fd=16, events=....是一直在等待http服务端返回数据，这解释了为什么进程处于busy状态 解决： 知道了进程阻塞在哪里，接下来就容易解决了，例如上面经过定位应该是业务在调用curl，而对应的url长时间没有返回数据，导致进程一直等待。这时候可以找url提供者定位url返回慢的原因，同时应该在curl调用的时候加上超时参数，比如2秒没返回就超时，避免长时间阻塞卡死(这样进程可能会出现2秒左右的busy状态)。