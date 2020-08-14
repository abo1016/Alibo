---
title: php 源码安装
tags: [编译安装]
id: '75'
categories:
    - linux
date: 2018-05-19 11:21:50
---

```
依赖库 apt-get install libxml2-dev autoconf libxml2-dev build-essential libcurl4-gnutls-dev openssl make 

1、下载源码（版本选择）
wget https://cn2.php.net/distributions/php-7.2.3.tar.gz 

wget https://cn2.php.net/distributions/php-7.2.0.tar.gz

wget https://cn2.php.net/distributions/php-7.1.15.tar.gz

wget https://cn.php.net/distributions/php-5.6.34.tar.gz

2解压
tar -zxvf php-7.2.3.tar.gz

3、准备安装文件夹
mkdir /usr/local/php

4、配置

cd php-7.2.3

A、如果PHP搭配Apache使用，那么配置如下
./configure –prefix=/usr/local/php –with-apxs2=/usr/local/apache2/bin/apxs

注：
/usr/local/apache2/bin/apxs，其中apxs是在安装Apache时产生的，apxs是一个为Apache HTTP服务器编译和安装扩展模块的工具，使之可以用由mod_so提供的LoadModule指令在运行时加载到Apache服务器中

B、如果只是单独安装PHP以及MySQL的扩展，而不安装MySQL服务，那么需要添加下面的配置
–enable-sockets=shared  –with-pdo-mysql=shared,mysqlnd 或者 –with-mysql=shared,mysqlnd

C、启动配置php-fpm
–enable-fpm

总结：
安装的是不带Apache 和 Mysql 服务器，并且使用PDO扩展，那么配置如下：
./configure –prefix=/usr/local/php –enable-sockets=shared  –enable-fpm –with-pdo-mysql=shared,mysqlnd

安装的带apache的：

./configure –prefix=/usr/local/php –with-apxs2=/usr/local/apache2/bin/apxs –enable-sockets=shared  –enable-fpm –with-pdo-mysql=shared,mysqlnd

5，编译 make
6、安装 make install 

注：如果过程失败，make distclean 并重新操作4 5 6 ，

7、复制ini
cp /php-7.2.0/php.ini-development  /usr/local/php/lib/php.ini
    把原来位于源代码里面的php.ini-development拷贝到/usr/local/php/lib/php.ini下，并且重命名为php.ini

8、把php加入到系统环境变量
echo “export PATH=$PATH:/usr/local/php/bin/php”  >> /etc/profile
source /etc/profile

9、查看php版本
/usr/local/php/bin/php –version

10、安装扩展
首先，请确保已经安装了autoconf，如未安装，请执行apt-get install autoconf
编译完成之后，将会自动把mysql.so放到了默认的php扩展目录下（phpinfo可查看，我的为 /usr/local/php/lib/php/extensions/no-debug-zts-20090626），再修改php.ini
修改php.ini,添加一句extension=mbstring.so

mbstring扩展
[html] view plain copy
1、进入源码mbstring文件夹  
cd /php-7.2.0/ext/mbstring  
2、/usr/local/php/bin/phpize   生成makefile文件  
3、执行生成configure（假设php安装在/usr/local/php目录下）  ./configure –with-php-config=/usr/local/php/bin/php-config  
4、编译&安装  
make && make install  
php.ini 添加一行：extension=mbstring.so

pdo_mysql扩展
1、进入源码pdo_mysql文件夹 
cd /php-7.2.0/ext/pdo_mysql 
2、执行生成configure（假设php安装在/usr/local/php目录下） 
/usr/local/php/bin/phpize 
3、生成makefile文件 
./configure –with-php-config=/usr/local/php/bin/php-config 
假如你在本地安装了mysql服务，那么需执行下面命令 
./configure –with-php-config=/usr/local/php/bin/php-config –with-pdo-mysql=/usr/local/mysql/ 
4、编译&安装 
make && make install 
5、修改php.ini,添加一句extension=pdo_mysql.so 

zlib扩展

此扩展进入源码/php-7.2.0/ext/zlib安装会出错，因此先执行下面语句 
1、 https://www.zlib.net/下载zlib源码 
wget https://www.zlib.net/zlib-1.2.11.tar.gz 
2、解压，配置，编译，安装 
tar -zxvf zlib-1.2.11.tar.gz 
cd zlib-1.2.11/ 
./configure –prefix=/usr/local/zlib 
make && make install 
3、重新配置、编译、安装PHP,增加参数–with-zlib-dir=/usr/local/zlib 
./configure –prefix=/usr/local/php \ 
–enable-sockets=shared \ 
–with-pdo-mysql=shared,mysqlnd \ 
–with-zlib-dir=/usr/local/zlib 

curl扩展
[html] view plain copy
1、安装apt-get install libcurl4-gnutls-dev，如果出错，请先apt-get update 
2、进入源码curl文件夹 
cd /php-7.2.0/ext/curl 
3、执行生成configure（假设php安装在/usr/local/php目录下） 
/usr/local/php/bin/phpize 
4、生成makefile文件 
./configure –with-php-config=/usr/local/php/bin/php-config 
5、编译&安装 
make && make install 
6、修改php.ini,添加一句extension=curl.so 

libevent/event 扩展

1. 由于PHP5.7以后只支持event，因此我安装的event，但是libevent的安装方法和event方法一样 
2. 扩展依赖于原始的libevent库，必须先把libevent库安装 
3. 安装libevent库(https://libevent.org/) 

1. wget https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz 

1. tar -zxvf libevent-2.1.8-stable.tar.gz 

1. cd libevent-2.1.8-stable/ 

1. ./configure –prefix=/usr/local/libevent-2.1.8/ 

1. make && make install 

1. 安装event扩展(https://pecl.php.net/package/event) 

1. wget https://pecl.php.net/get/event-2.3.0.tgz 

1. tar -zxvf event-2.3.0.tgz 

1. cd event-2.3.0/ 

1. /usr/local/php/bin/phpize 

1. ./configure –with-php-config=/usr/local/php/bin/php-config –with-event-libevent-dir=/usr/local/libevent-2.1.8/ 

1. 如果是libevent

1. ./configure –with-php-config=/usr/local/php/bin/php-config –with-libevent=/usr/local/libevent-2.1.8/ 

1. make && make install 

redis扩展(phpredis)
1、下载源码https://github.com/phpredis/phpredis/releases 
wget https://github.com/phpredis/phpredis/archive/3.1.4.tar.gz 
2、mv 3.1.4.tar.gz phpredis.tar.gz 
3、tar -zxvf phpredis.tar.gz 
4、cd phpredis-3.1.4/ 
5、/usr/local/php/bin/phpize 
6、./configure –with-php-config=/usr/local/php/bin/php-config 
7、 make && make install 
8、echo extension=redis.so >> /usr/local/php/lib/php.ini

openssl扩展
1、进入源码openssl文件夹 
cd /php-7.2.0/ext/openssl 
2、执行生成configure（假设php安装在/usr/local/php目录下） 
cp config0.m4 config.m4 
/usr/local/php/bin/phpize 
3、生成makefile文件 
./configure –with-php-config=/usr/local/php/bin/php-config 
4、编译&安装 
make && make install 
5、echo extension=openssl.so >> /usr/local/php/lib/php.ini
```