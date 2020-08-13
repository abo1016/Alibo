---
title: python  uWSGI、WSGI和uwsgi
tags: []
id: '482'
categories:
  - - all-blog
    - python
date: 2019-07-18 11:59:08
---

### 我们在理解了PHP的运行方式和**php-fpm、fastcgi、cgi**等这些概念之后，在学习python时也会想起，Python作为和PHP一样的不需要编译的**动态解释型语言**在工作时的方式，是不是有类似于和PHP那些软件和协议呢！下面内容来学习一下Python中的uWSGI、WSGI和uwsgi。

**首先来看几张图片：** ![](https://blog.wenboo.top/wp-content/uploads/2019/07/cd4b873f4d5447c91bd4d97d10f28b0d.png) ![](https://blog.wenboo.top/wp-content/uploads/2019/07/967ecfd077e1c4077f8b210de5da49ea.png) 图片来自：[https://www.cnblogs.com/wspblog/p/8575101.html](https://www.cnblogs.com/wspblog/p/8575101.html "https://www.cnblogs.com/wspblog/p/8575101.html")

### WSGI(Web Server Gateway Interface)

从字面上我知道这个东西叫做**web服务器网关接口**，联想到PHP的php-cgi是一种web服务器和应用程序进行交互的协议或接口，WSGI我把它理解为python版的fastcig，它实现了标准的CGI协议。**WSGI是Web 服务器(uWSGI)与 Web 应用程序或应用框架(Django)之间的一种低级别的接口**。

### uWSGI

它和nginx、Apache一样是一个web服务器，并且它能够将HTTP协议转换成其他协议，比如转换成WSGI和python应用程序交互。但是有个疑问？uWSGI已经是web服务器了为什么我们常见的架构还需要Apache和nginx这类的服务器软件参与，因为uWSGI运行效率太低，性能差直接使用uWSGI来搭建web服务器扛不了并发，而且像nginx这类web服务器支持反向代理，做负载均衡水平拓展非常方便。

### uwsgi

与WSGI一样，是uWSGI服务器的独占通信协议，用于定义传输信息的类型(type of information)。每一个uwsgi packet前4byte为传输信息类型的描述，与WSGI协议是两种东西，据说该协议是fcgi协议的10倍快。

### Django、Tornado

这些都是**python**的web开发框架，它们之所以能够直接接收HTTP协议，是因为内置了轻量级的web服务器，但是性能和安全方面不能够保证，官方建议不要在生产环境使用。