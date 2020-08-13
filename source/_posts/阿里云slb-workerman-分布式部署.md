---
title: 阿里云SLB + workerman 分布式部署
tags: []
id: '32'
categories:
  - - all-blog
    - nginx
date: 2018-05-11 10:47:38
---

近两天工作上的任务，要在云端加一台服务器将workerman 做成分布式集群，现在做一下总结吧。首先购买阿里SLB和ECS就略过了，阿里的文档已经写的很清楚了。 重点记录一下遇到的坑吧，首先是源码安装nginx的时候1.8.1版本有个官方bug，与opensll-1.1.0g不兼容，死活make不通过，换nginx最新稳定版解决 以下是源码安装nginx的一些步骤： 环境：ubuntu 14.04 （Apahce + php + nginx + workerman + redis + mysql ）nginx做反向代理http&&https 1、Nginx源码下载 https://nginx.org/download/nginx-(版本号)tar.gz 2、下载依赖模块pcre、openssl、zlib 2.1、pcre：[https://www.pcre.org/](https://www.pcre.org/) wget [https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.gz](https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.gz) 2.2、openssl:https://www.openssl.org/ wget https://www.openssl.org/source/openssl-1.1.0g.tar.gz 2.3、zlib:https://zlib.net wget：https://zlib.net/zlib-1.2.11.tar.gz 3、安装前配置

```shell
./configure –sbin-path=/usr/local/nginx/nginx \
–conf-path=/usr/local/nginx/nginx.conf \
–pid-path=/usr/local/nginx/nginx.pid \
–with-http_ssl_module \
–with-pcre=/root/pcre-8.41 \
–with-zlib=/root/zlib-1.2.11 \
–with-openssl=/root/openssl-1.1.0g
```

4、编译和安装 `make && make install` 在/etc/init.d/ 添加开机脚本nginx

```shell
#!/bin/bash
### BEGIN INIT INFO
# Provides: Nginx
# Required-Start: $network $remote_fs $syslog
# Required-Stop: $network $remote_fs $syslog
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Very secure Nginx HTTP server
# Description: Nginx is a high-performance web and proxy server
### END INIT INFO
```

#注意：这里的三个变量需要根据具体的环境而做修改。

```shell
nginxd=/usr/local/nginx/nginx
nginx_config=/usr/local/nginx/nginx.conf
nginx_pid=/usr/local/nginx/logs/nginx.pid
RETVAL=0
prog=”nginx”
```

```shell
# Check that networking is up.
[ -x $nginxd ] || exit 0
# Start nginx daemons functions.
start() {
if [ -e $nginx_pid ];then
echo “nginx already running….”
exit 1
fi
echo -n $”Starting $prog: ”
$nginxd -c ${nginx_config}
RETVAL=$?
echo
[ $RETVAL = 0 ]
return $RETVAL
}
# Stop nginx daemons functions.
stop() {
echo -n $”Stopping $prog: ”
$nginxd -s stop
RETVAL=$?
echo
[ $RETVAL = 0 ] && rm -f /var/lock/subsys/nginx $nginx_pid
}
# reload nginx service functions.
reload() {
echo -n $”Reloading $prog: ”
kill -HUP `cat ${nginx_pid}`
RETVAL=$?
echo
}
# See how we were called.
case “$1″ in
start)
start
;;
stop)
stop
;;
reload)
reload
;;
restart)
stop
start
;;
status)
status $prog
RETVAL=$?
;;
*)
echo $”Usage: $prog {start|stop|restart|reload|status|help}”
exit 1
esac
exit $RETVAL
```

设置脚本权限 `chmod -R 755 /etc/init.d/nginx` 加入开机服务 `update-rc.d nginx defaults` 可以使用 sysv-rc-conf工具进行查看和管理开机启动项（需要安装apt-get install sysv-rc-conf） 启动、重启、停止 `service nginx start|stop|restart` 平滑重启 `kill -USR2 pid`

> workerman 部分：

使用的是GatewayWorker 做集群，选择一台master服务器提供Register服务（端口1234）即可（统计socket时直接使用这台服务器ip）假设这台服务器为192.168.1.1，其他服务器就不需要 start\_register.php这个文件了，然后将start\_geteway.php、start\_businessworker.php中的registerAddress为192.168.1.1:1234，start\_gateway.php中的lanI为当前服务器的内网ip：192.168.1.1（所有服务器的ip都要是内网或者外网ip，一定不能是回路ip 127.0.0.1） 其他task服务器只需配置start\_gateway ，registerAddress为192.168.1.1:1234,，lanip为本服务器ip。start\_businessworker.php的registerAddress:192.168.1.1:1234 ①、Register服务监听的端口要可以被其它内网服务器访问（外网访问可以屏蔽）； ②、start\_gateway.php中如果$gateway->startPort=2300; $gateway->count=4;，则2300 2301 2302 2303四个端口需要被设置成能被其它服务器访问，也就是起始端口$gateway->startPort到 $gateway->startPort + $gateway->count - 1这 $gateway->count个端口要设置成能被其它内网 服务器访问。(在阿里云控制台安全组规则允许入网 ，可以debug模式运行端口通讯是否正常)在前端使 用SLB进行请求接收和转发给，完成部署。