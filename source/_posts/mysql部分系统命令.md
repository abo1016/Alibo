---
title: mysql部分系统命令
tags: [mysql, 命令]
id: '104'
categories:
    - mysql
date: 2018-05-25 23:47:31
---

1、查看所有系统设置  
show variables;  
  
  
2、查看指定系统设置  
show variables like '%bulk\_insert\_buffer\_size%'  
  
  
3、设置  
set low\_priority\_updates=0;  
  
  
4、常见系统参数  
show status like 'uptime';  
show status like 'com\_select';  
show status like 'com\_insert';  
show status like 'com\_update';  
show status like 'com\_delete';  
show status like 'connections';  
show status like 'slow\_queries';  
  
  
5、查看MySQL连接数  
show processlist  
show full processlist  
  
  
或者   
mysqladmin -uroot -p processlist  
  
  
6、只查看当前连接数(Threads就是连接数.):  
mysqladmin  -uadmin -p status  
  
  
7、设置mysql链接数据  
#vim /etc/my.cnf  
max\_user\_connections=30 这个就是单用户的连接数  
max\_connections=800 这个是全局的限制连接数