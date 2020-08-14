---
title: 初识 swoole
tags: [swoole]
id: '1'
categories:
    - PHP
date: 2018-04-10 22:48:55
---

> PHP作为“世界上最好的语言”，一直以来被广用于web程序开发，仅此而已。它来了– Swoole：面向生产环境的 PHP 异步网络通信引擎。 PHP的异步、并行、高性能网络通信引擎，使用纯C语言编写，提供了PHP语言的异步多线程服务器，异步TCP/UDP网络客户端，异步MySQL，异步Redis，数据库连接池，AsyncTask，消息队列，毫秒定时器，异步文件读写，异步DNS查询。 Swoole内置了Http/WebSocket服务器端/客户端、Http2.0服务器端/客户端（官网复制）。

反正swoole各种牛逼就是了哈哈哈 :smile: 竟然这么牛逼我们就开始安装一步一步来学习它吧，在安装之前我们要先确认环境(只讲linux下)是否符合：

> 仅支持Linux、FreeBSD、MacOS三种操作系统

在Windows平台，可使用CygWin或WSL(Windows Subsystem for Linux) Linux内核版本2.3.32以上 gcc4.4以上版本或者clang 编译为libswoole.so作为C/C++库时需要使用cmake-2.4或更高版本 建议使用Ubuntu14、CentOS7或更高版本的操作系统

> PHP版本依赖

Swoole-1.x需要PHP-5.3.10或更高版本 Swoole-2.x需要PHP-7.0.0或更高版本 不依赖PHP的stream、sockets、pcntl、posix、sysvmsg等扩展。PHP只需安装最基本的扩展即可

> 编译安装

Swoole扩展是按照PHP标准扩展构建的。使用phpize来生成编译检测脚本，./configure来做编译配置检测，make进行编译，make install进行安装。 请下载releases版本的swoole，直接从github主干上拉取最新代码可能会编译不过 如果当前用户不是root，可能没有PHP安装目录的写权限，安装时需要sudo或者su 如果是在git分支上直接git pull更新代码，重新编译前务必要执行make clean 下载地址 https://github.com/swoole/swoole-src/releases https://pecl.php.net/package/swoole https://git.oschina.net/swoole/swoole 下载源代码包后，在终端进入源码目录，执行下面的命令进行编译和安装

```
cd swoole
phpize
./configure
make 
sudo make install
```

PECL swoole项目已收录到PHP官方扩展库，除了手工下载编译外，还可以通过PHP官方提供的pecl命令，一键下载安装swoole `pecl install swoole` 配置php.ini 编译安装成功后，修改php.ini加入 `extension=swoole.so` 通过php -m或phpinfo()来查看是否成功加载了swoole，如果没有可能是php.ini的路径不对，可以使用php –ini来定位到php.ini的绝对路径。

> 快速起步

Swoole的绝大部分功能只能用于cli命令行环境，请首先准备好Linux Shell环境。可使用vim、emacs、phpstorm或其他编辑器编写代码，并在命令行中通过下列指令执行程序。 `php /path/to/your_file.php` 成功执行Swoole服务器程序后，如果你的代码中没有任何echo语句，屏幕不会有任何输出，但实际上底层已经在监听网络端口，等待客户端发起连接。可使用相应的客户端工具和程序连接到服务器，进行测试。

> 进程管理

默认使用SWOOLE\_PROCESS模式，因此会额外创建Master和Manager两个进程。在设置worker\_num之后，实际会出现2 + worker\_num个进程 服务器启动后，可以通过kill 主进程ID来结束所有工作进程

> PHP环境

Swoole提供的绝大的部分模块只能用于cli命令行终端。目前只有Client同步客户端可以用于php-fpm环境下。请勿在Web环境中使用Server等模块。