---
title: tomcat部署war
tags: [tomcat, war]
id: '341'
categories:
    - java
date: 2018-11-16 18:07:55
---

最近做Google assistant actions中的Implement Report State api 调试工具（[github](https://github.com/actions-on-google/smart-home-dashboard "github")）官方只给了java环境的tool,记录一下搭建java web环境。

### 安装java环境

去官网下载jdk最新版，[下载地址](https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html "下载地址")。 新建文件：`mkdir java_eno`，将下载好的jdk文件解压到此。 配置环境变量：

```shell
export JAVA_HOME=/root/javaeno/jdk-11.0.1
export JAVA_BIN=/root/javaeno/jdk-11.0.1/bin
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME JAVA_BIN PATH CLASSPATHSPH
```

### 安装tomcat

官方下载tomcat [下载地址](http://tomcat.apache.org/ "下载地址")解压，并进入目录。 将dash.war项目文件拷贝到webapps文件夹下，配置./conf/server.xml在Host标签内添加以下语句。

```markup
<Context path="/" docBase="dash" debug="0" reloadable="true" privileged="true"/>
```

启动tomcat进程

```shell
cd ../bin
sh startup.sh
```

出现这样界面说明成功

```shell
Using CATALINA_BASE:   /root/javaeno/apache-tomcat-9.0.13
Using CATALINA_HOME:   /root/javaeno/apache-tomcat-9.0.13
Using CATALINA_TMPDIR: /root/javaeno/apache-tomcat-9.0.13/temp
Using JRE_HOME:        /root/javaeno/jdk-11.0.1
Using CLASSPATH:       /root/javaeno/apache-tomcat-9.0.13/bin/bootstrap.jar:/root/javaeno/apache-tomcat-9.0.13/bin/tomcat-juli.jar
Tomcat started.

```

[![ixrywT.md.png](https://s1.ax1x.com/2018/11/16/ixrywT.md.png)](https://imgchr.com/i/ixrywT)