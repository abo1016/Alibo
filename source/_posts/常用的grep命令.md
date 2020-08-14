---
title: 常用的grep命令
tags: [grep]
id: '50'
categories:
    - linux
    - shell
date: 2018-05-15 09:53:37
---

**grep (global search regular expression(RE) and print out the line,全面搜索正则表达式并把行打印出来)是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。**

## **grep常用用法**

\[root@www ~\]# grep \[-acinv\] \[--color=auto\] '搜寻字符串' filename
选项与参数：
-a ：将 binary 文件以 text 文件的方式搜寻数据
-c ：计算找到 '搜寻字符串' 的次数
-i ：忽略大小写的不同，所以大小写视为相同
-n ：顺便输出行号
-v ：反向选择，亦即显示出没有 '搜寻字符串' 内容的那一行！
--color=auto ：可以将找到的关键词部分加上颜色的显示喔！

  

取出/etc/passwd 有出现root的行

root@vagrant-ubuntu-trusty-64:# grep root /etc/passwd  
root:x:0:0:root:/root:/bin/bash

显示行号

root@vagrant-ubuntu-trusty-64:# grep -n root /etc/passwd  
1:root:x:0:0:root:/root:/bin/bash

  

将/etc/passwd，将没有出现 root 的行取出来  

grep -v root /etc/passwd

  

daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin  
bin:x:2:2:bin:/bin:/usr/sbin/nologin  
sys:x:3:3:sys:/dev:/usr/sbin/nologin  
sync:x:4:65534:sync:/bin:/bin/sync  
games:x:5:60:games:/usr/games:/usr/sbin/nologin  
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin  
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin  
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin  
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin  
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin  
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin  
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin  
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin  
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin  
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin  
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin  
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin  
libuuid:x:100:101::/var/lib/libuuid:  
syslog:x:101:104::/home/syslog:/bin/false  
messagebus:x:102:106::/var/run/dbus:/bin/false  
landscape:x:103:109::/var/lib/landscape:/bin/false  
sshd:x:104:65534::/var/run/sshd:/usr/sbin/nologin  
pollinate:x:105:1::/var/cache/pollinate:/bin/false  
vagrant:x:1000:1000::/home/vagrant:/bin/bash  
colord:x:106:112:colord colour management daemon,,,:/var/lib/colord:/bin/false  
statd:x:107:65534::/var/lib/nfs:/bin/false  
puppet:x:108:114:Puppet configuration management daemon,,,:/var/lib/puppet:/bin/false  
ubuntu:x:1001:1001:Ubuntu:/home/ubuntu:/bin/bash  
mysql:x:999:1002::/home/mysql:/bin/false

  

  

用 dmesg 列出核心信息，再以 grep 找出内含 eth 那行,要将捉到的关键字显色，且加上行号来表示：

\[root@www ~\]# dmesg | grep -n --color=auto 'eth'
247:eth0: RealTek RTL8139 at 0xee846000, 00:90:cc:a6:34:84, IRQ 10
248:eth0: Identified 8139 chip type 'RTL-8139C'
294:eth0: link up, 100Mbps, full-duplex, lpa 0xC5E1
305:eth0: no IPv6 routers present
# 你会发现除了 eth 会有特殊颜色来表示之外，最前面还有行号喔！

在关键字的显示方面，grep 可以使用 --color=auto 来将关键字部分使用颜色显示。 这可是个很不错的功能啊！但是如果每次使用 grep 都得要自行加上 --color=auto 又显的很麻烦～ 此时那个好用的 alias 就得来处理一下啦！你可以在 ~/.bashrc 内加上这行：『alias grep='grep --color=auto'』再以『 source ~/.bashrc 』来立即生效即可喔！ 这样每次运行 grep 他都会自动帮你加上颜色显示

  

  

  

用 dmesg 列出核心信息，再以 grep 找出内含 eth 那行,在关键字所在行的前两行与后三行也一起捉出来显示

\[root@www ~\]# dmesg | grep -n -A3 -B2 --color=auto 'eth'
245-PCI: setting IRQ 10 as level-triggered
246-ACPI: PCI Interrupt 0000:00:0e.0\[A\] -> Link \[LNKB\] ...
247:eth0: RealTek RTL8139 at 0xee846000, 00:90:cc:a6:34:84, IRQ 10
248:eth0: Identified 8139 chip type 'RTL-8139C'
249-input: PC Speaker as /class/input/input2
250-ACPI: PCI Interrupt 0000:00:01.4\[B\] -> Link \[LNKB\] ...
251-hdb: ATAPI 48X DVD-ROM DVD-R-RAM CD-R/RW drive, 2048kB Cache, UDMA(66)
# 如上所示，你会发现关键字 247 所在的前两行及 248 后三行也都被显示出来！
# 这样可以让你将关键字前后数据捉出来进行分析啦！

  

  

  

根据文件内容递归查找目录

\# grep ‘energywise’ \*           #在当前目录搜索带'energywise'行的文件

# grep -r ‘energywise’ \*        #在当前目录及其子目录下搜索'energywise'行的文件

\# grep -l -r ‘energywise’ \*     #在当前目录及其子目录下搜索'energywise'行的文件，但是不显示匹配的行，只显示匹配的文件

这几个命令很使用，是查找文件的利器。

grep是一个无比强大的命令还有许多更加高阶的正则用法