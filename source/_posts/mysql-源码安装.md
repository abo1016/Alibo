---
title: mysql 源码安装
tags: []
id: '77'
categories:
  - - all-blog
    - linux
date: 2018-05-19 11:23:17
---

**依赖：** `apt-get install make bison g++ build-essential libncurses5-dev cmake sysv-rc-conf git` **1、添加mysql用户组并添加mysql用户，并且不允许登录**

```
groupadd mysql
useradd -r -g mysql -s /bin/false -M mysql
```

**2、下载mysql源码包，此处下载为mysql5.7.20** `https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.20.tar.gz` **解压**

```
tar -zxvf mysql-5.7.20.tar.gz
cd mysql-5.7.20
```

**3、更新环境**

```
apt-get update
apt-get upgrade
```

**安装gcc、bison、cmake**

```
apt-get install gcc
apt-get install bison
apt-get install cmake
```

**4、安装** 4.1、创建安装路径以及数据存储路径

```
mkdir /usr/local/mysql
mkdir /usr/local/mysql/data
```

4.2、cmake配置

```
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock \
-DSYSCONFDIR=/etc \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DDOWNLOAD_BOOST=1 \
-DWITH_BOOST=/usr/local/mysql/mysql-boost \
-DMYSQL_TCP_PORT=3306 \
-DENABLE_DOWNLOADS=1

错误1：ubuntu mysql Could NOT find Git
apt-get install git
另外需要rm CMakeCache.txt
错误2：No CMAKE_CXX_COMPILER could be found
apt-get install g++
错误3：Curses library not found
apt-get install libncurses5-dev
另外需要rm CMakeCache.txt
错误4：g++: internal compiler error: Killed (program cc1plus)
主要原因大体上是因为内存不足，临时使用交换分区来解决
sudo dd if=/dev/zero of=/swapfile bs=64M count=16
sudo mkswap /swapfile
sudo swapon /swapfile
安装完成之后，删除创建的交换分区
sudo swapoff /swapfile
sudo rm /swapfile
```

**4.3、make && make install** **5、权限限定** `chown -R mysql: /usr/local/mysql` **6、初始化数据库**

```
/usr/local/mysql/bin/mysqld –initialize –user=mysql –basedir=/usr/local/mysql –datadir=/usr/local/mysql/data

/usr/local/mysql/bin/mysqld –initialize-insecure –user=mysql –basedir=/usr/local/mysql/ –datadir=/usr/local/mysql/data/ 
```