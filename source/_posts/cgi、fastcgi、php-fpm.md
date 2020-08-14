---
title: CGI、FastCGI、PHP-FPM
tags: [CGI, FastCGI, PHP-FPM]
id: '228'
categories:
    - PHP
date: 2018-07-12 15:36:11
---

转：[文章出处](https://www.cnblogs.com/f-ck-need-u/p/7627035.html "https://www.cnblogs.com/f-ck-need-u/p/7627035.html")

> CGI -- common gateway interface的缩写，通常译为：通用网关接口

相信多数人和我一样无法知其名而知其意，cgi它是一种协议。通过cgi协议，web server可以将动态请求和相关参数发送给专门处理动态内容的应用程序。 web服务器所处理的内容都是静态的，要想处理动态内容，需要依赖于web应用程序，如php、jsp、python、perl等。web服务器要将请求交给web应用程序就必须要通过cgi协议的帮助，web server和web应用交换信息的规范。

> 简单cgi工作图

[![](https://i.loli.net/2018/07/12/5b46d545ed88d.png)](https://i.loli.net/2018/07/12/5b46d545ed88d.png)

> 术语释疑

不是科班出身的同学就像我，接触编程时都会被一些专业术语名词搞出一堆疑问，cgi到底是个什么鬼，通用网关接口又是个啥，fast-cgi呢？ 以下是以php为例各名词解释 **cgi**：它是一种协议。通过cgi协议，web server可以将动态请求和相关参数发送给专门处理动态内容的应用程序。 **fast-cgi**：也是一种协议，只不过是cgi的优化版。cgi的性能较烂，fastcgi则在其基础上进行了改进。 **php-cgi**：fastcgi是一种协议，而php-cgi实现了这种协议。不过这种实现比较烂。它是单进程的，一个进程处理一个请求，处理结束后进程就销毁。 **php-fmp**：是对php-cgi的改进版，它直接管理多个php-cgi进程/线程。也就是说，php-fpm是php-cgi的进程管理器因此它也算是fastcgi协议的实现。在一定程度上讲，php-fpm与php的关系，和tomcat对java的关系是类似的。 **cgi进程/线程**：在php上，就是php-cgi进程/线程。专门用于接收web server的动态请求，调用并初始化zend虚拟机。 **cgi脚本**：被执行的php源代码文件。 **zend虚拟机**：对php文件做词法分析、语法分析、编译成opcode，并执行。最后关闭zend虚拟机。 **cgi进程/线程和zend虚拟机的关系**：cgi进程调用并初始化zend虚拟机的各种环境。 以php-fpm为例，web server从转发动态请求到结束的过程大致如下： [![](https://i.loli.net/2018/07/12/5b46f288cdb4e.png)](https://i.loli.net/2018/07/12/5b46f288cdb4e.png) 而每个php-cgi进程的作用大致包括：(有些功能分类错误，请无视，知道大致功能就够了) [![](https://i.loli.net/2018/07/12/5b46f3dbd59c0.png)](https://i.loli.net/2018/07/12/5b46f3dbd59c0.png)

> web server和CGI的交互模式

web server对cgi进程/线程来说，它的作用就是发起动态处理请求，传递一些参数和环境变量，最后接收cgi的返回结果。再通俗而不严谨地说，web server通过cgi/fastcgi协议将动态请求转发给执行cgi脚本的应用程序。 以最典型的apache httpd和php为例，对于httpd来说，web server和php-cgi有3种交互模式。 **cgi模式**：httpd接收到一个动态请求就fork一个cgi进程，cgi进程返回结果给httpd进程后自我销毁。 **动态模块模式**：将php-cgi的模块(例如php5\_module)编译进httpd。在httpd启动时会加载模块，加载时也将对应的模块激活，php-cgi也就启动了。(注：纠正一个小小错误，很多人以为动态编译的模块是可以在需要的时候随时加载调用，不需要的时候它们就停止了，实际上不是这样的。和静态编译的模块一样，动态加载的模块在被加载时就被加入到激活链表中，无论是否使用它，它都已经运行在apache httpd的内部。可参考LoadModule指令的官方手册) **php-fpm模式**：使用php-fpm管理php-cgi，此时httpd不再控制php-cgi进程的启动。可以将php-fpm独立运行在非web服务器上，实现所谓的动静分离。 实际上，借助模块mod\_fastcgi还可以实现fastcgi模式。同cgi一样，管理模式的先天缺陷决定了这并不是一种好方法。

> CGI模式

使用CGI模式时，当动态请求到达，httpd临时启动一个cgi解释器，并通过cgi协议转发要运行的内容。当cgi脚本运行结束后，将结果返回给httpd，然后cgi解释器进程自我销毁。当多个动态请求到达时，将先后启动多个cgi解释器。因此，这种方法效率极低。

> 模块方式

在编译php时，将php5\_module模块编译到apache中，例如在编译php时在./configure配置中加上"--with-apxs2=/usr/local/apache/bin/apxs"。 这种交互模式下，httpd在启动时加载并激活php\_module。也就是说，php-cgi常驻在httpd进程内部。当动态请求到达时，httpd不用再生成cgi解释器，而是直接将动态请求转发给它内部php-cgi。 配置实用这种交互模式非常简单，只需使用LoadModule加载php\_module，再添加对应的MIME处理器即可。

```shell
LoadModule php5_module modules/libphp5.so

# 在mime模块中添加对应的类型
<IfModule mime_module>
AddType application/x-httpd-php .php
AddType applicaiton/x-httpd-php-source .phps
</IfModule>
```

> php-fpm方式

前面说了，php-fpm是php-cgi的进程管理器。这种交互方式实际上是让php-cgi以独立于httpd的方式存在，目前基本使用php-fpm的方式管理php-cgi进程。也就是说，这种模式下，php-cgi和httpd已经分离了，它们的分离意味着请求的动静分离变为可能：httpd和php-fpm分别运行在不同服务器上。动静分离后，压力也分散到各自的服务器上。 要让php-fpm以这种方式运行，需要在编译的./configure配置选项中添加"--enable-fpm"选项。当然，还得启动php-fpm服务。例如： `service php-fpm start` 这样php-cgi进程就开放着端口(默认9000)等待httpd转发动态请求。要让httpd能够转发请求到php-cgi上，需要在httpd.conf中关闭正向代理，并设置fastcgi协议代理参数。例如，转发到192.168.100.54主机上的php-fpm。

```shell
# 加载代理模块
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so

# 添加MIME类型
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps

# 在需要转发的虚拟主机中配置转发代理
ProxyRequests off
ProxyPassMatch ^/(.*\.php)$ fcgi://192.168.100.54:9000/usr/local/apache/htdocs/$1
```