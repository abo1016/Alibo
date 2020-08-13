---
title: apache 源码安装
tags: []
id: '73'
categories:
  - - all-blog
    - linux
date: 2018-05-19 11:20:13
---

  
  
注：apt-get install  gcc  make  g++  build-essential   
  
1、wget https://mirrors.tuna.tsinghua.edu.cn/apache/httpd/httpd-2.4.32.tar.gz https://mirrors.tuna.tsinghua.edu.cn/apache//apr/apr-1.6.3.tar.gz https://mirrors.tuna.tsinghua.edu.cn/apache//apr/apr-util-1.6.1.tar.gz https://ftp.pcre.org/pub/pcre/pcre-8.40.tar.gz  
  
2、tar -zxvf apr-1.6.3.tar.gz  && tar -zxvf apr-util-1.6.1.tar.gz && tar -zxvf httpd-2.4.32.tar.gz && tar -zxvf  pcre-8.40.tar.gz  
  
3、建立目录  
 mkdir /usr/local/apache2  
  
先把apr及apr-util下载解压到apache文件夹中的子文件夹srclib的apr及apr-util中，在进行apache的安装  
  
cp -r apr-1.6.3 httpd-2.4.32/srclib/apr && cp -r apr-util-1.6.1 httpd-2.4.32/srclib/apr-util  
  
4、安装pcre  
cd pcre-8.40   
mkdir /usr/local/pcre  
./configure --prefix=/usr/local/pcre  
make  
make install  
  
5 、配置  
  
cd ../httpd-2.4.32   
A、假如apr和apr-util是拷贝到httpd-2.4.23/srclib中，那么只需要  
./configure --prefix=/usr/local/apache2 --with-pcre=/usr/local/pcre --enable-module=shared  
  
B、假如apr和apr-util都是和pcre那样进行编译安装的，因此需要如下配置  
./configure --prefix=/usr/local/apache2 --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --with- pcre=/usr/local/pcre --enable-module=shared  
  
5、编译  
    make      
6、安装  
    make install  
  
7、vi /usr/local/apache2/conf/httpd.conf   
  
  找到：  
    AddType  application/x-compress .Z  
    AddType application/x-gzip .gz .tgz  
    在后面添加：  
    AddType application/x-httpd-php .php（使Apcche支持PHP）  
    AddType application/x-httpd-php-source .phps（phps支持php7）  
  
  找到：  
    <IfModule dir\_module>  
    DirectoryIndex index.html  添加 index.php  
    </IfModule>   
  
 找到：  
    ＃ServerName www.example.com:80  
    修改为：  
    ServerName 127.0.0.1:80或者ServerName localhost:80  
  
  
8、加入到系统开机自动启动服务器  
cp /usr/local/apache2/bin/apachectl /etc/init.d/apache2  
sysv-rc-conf apache2 on   
或者 #update-rc.d apache2  defaults  
查看  
sysv-rc-conf  
sysv-rc-conf --list  
sysv-rc-conf --list apache2    
注意：2345空格开启  
  
9、apache2.启动，重启和停止 ，先切换到安装完成后的目录/usr/local/apache2/bin  
    ./apachectl -k start  
    ./apachectl -k restart  
    ./apachectl -k stop