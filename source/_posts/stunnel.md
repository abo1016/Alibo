---
title: Stunnel
tags: [代理, tcp, ssl]
id: '445'
categories:
    - linux
date: 2019-05-23 11:01:26
---

**很高兴！！说的通过Amazon alexa 音箱来播放IPC视频的目标已经实现了。**

*   此次验证了当初通过RTSP redirect 的方案来实现此需求的可行性

> 先看效果

\[video width="544" height="960" mp4="https://blog.wenboo.top/wp-content/uploads/2019/05/VID\\\_20190523\\\_102529\\\_cps.mp4"\\\] 为了满足Amazon的条件，最后一步需要给RTSP通过CA证书的加密。RTSP不同于HTTP协议，无法通过nginx或者Apache等软件方便的引入cert和key进行ssl加密，我这次选择的是**Stunnel**

> Stunnel是一个自由的跨平台软件，用于提供全局的TLS/SSL服务。针对本身无法进行TLS或SSL通信的客户端及服务器，Stunnel可提供安全的加密连接。该软件可在许多操作系统下运行，包括Unix-like系统，以及Windows。Stunnel依赖于某个独立的库，如OpenSSL或者SSLeay，以实现TLS或SSL协议

ubuntu系统下通过`apt-get install Stunnel4`安装 安装完成后进入`cd /etc/stunnel`目录 `vi stunnel.conf` 原来这个目录是没有stunnel.conf这个文件的。我们可以通过`cat REDEME` 看到`/usr/share/doc/stunnel4/examples/stunnel.conf-sample`目录下有个示例文件

```shell
; Sample stunnel configuration file for Unix by Michal Trojnara 2002-2015
; Some options used here may be inadequate for your particular configuration
; This sample file does *not* represent stunnel.conf defaults
; Please consult the manual for detailed description of available options

; **************************************************************************
; * Global options                                                         *
; **************************************************************************

; It is recommended to drop root privileges if stunnel is started by root
;setuid = stunnel4
;setgid = stunnel4

; PID file is created inside the chroot jail (if enabled)
;pid = /var/run/stunnel.pid

; Debugging stuff (may be useful for troubleshooting)
;foreground = yes
;debug = info
;output = /var/log/stunnel.log

; Enable FIPS 140-2 mode if needed for compliance
;fips = yes

; **************************************************************************
; * Service defaults may also be specified in individual service sections  *
; **************************************************************************

; Enable support for the insecure SSLv3 protocol
;options = -NO_SSLv3

; These options provide additional security at some performance degradation
;options = SINGLE_ECDH_USE
;options = SINGLE_DH_USE

; **************************************************************************
; * Include all configuration file fragments from the specified folder     *
; **************************************************************************

;include = /etc/stunnel/conf.d

; **************************************************************************
; * Service definitions (remove all services for inetd mode)               *
; **************************************************************************

; ***************************************** Example TLS client mode services

; The following examples use /etc/ssl/certs, which is the common location
; of a hashed directory containing trusted CA certificates.  This is not
; a hardcoded path of the stunnel package, as it is not related to the
; stunnel configuration in /etc/stunnel/.

[gmail-pop3]
client = yes
accept = 127.0.0.1:110
connect = pop.gmail.com:995
verify = 2
CApath = @sysconfdir/ssl/certs
checkHost = pop.gmail.com
OCSPaia = yes

[gmail-imap]
client = yes
accept = 127.0.0.1:143
connect = imap.gmail.com:993
verify = 2
CApath = @sysconfdir/ssl/certs
checkHost = imap.gmail.com
OCSPaia = yes

[gmail-smtp]
client = yes
accept = 127.0.0.1:25
connect = smtp.gmail.com:465
verify = 2
CApath = @sysconfdir/ssl/certs
checkHost = smtp.gmail.com
OCSPaia = yes

; ***************************************** Example TLS server mode services

;[pop3s]
;accept  = 995
;connect = 110
;cert = /etc/stunnel/stunnel.pem

;[imaps]
;accept  = 993
;connect = 143
;cert = /etc/stunnel/stunnel.pem

;[ssmtp]
;accept  = 465
;connect = 25
;cert = /etc/stunnel/stunnel.pem

; TLS front-end to a web server
;[https]
;accept  = 443
;connect = 80
;cert = /etc/stunnel/stunnel.pem
; "TIMEOUTclose = 0" is a workaround for a design flaw in Microsoft SChannel
; Microsoft implementations do not use TLS close-notify alert and thus they
; are vulnerable to truncation attacks
;TIMEOUTclose = 0

; Remote shell protected with PSK-authenticated TLS
; Create "/etc/stunnel/secrets.txt" containing IDENTITY:KEY pairs
;[shell]
;accept = 1337
;exec = /bin/sh
;execArgs = sh -i
;ciphers = PSK
;PSKsecrets = /etc/stunnel/secrets.txt

; Non-standard MySQL-over-TLS encapsulation connecting the Unix socket
;[mysql]
;cert = /etc/stunnel/stunnel.pem
;accept = 3307
;connect = /run/mysqld/mysqld.sock

; vim:ft=dosini
```

可以直接拷贝过来然后修改，将不需要的内容都注释掉，最后在内容中增加了以下内容即可

```shell
cert = /etc/stunnel/rtsp.pem
key = /etc/stunnel/rtsp.key

; Some performance tunings
socket = l:TCP_NODELAY=1
socket = r:TCP_NODELAY=1

output = /var/log/stunnel.log

; Some debugging stuff useful for troubleshooting
;debug = 7
;foreground=yes


; Service-level configuration

[rtsp]
client = no
accept = 443
connect = 554 
TIMEOUTclose = 0
```

其中`[RTSP]`下面的内容为本次RTSP需要加密的配置 `client = no` no为服务端，yes为客户端 `accept` 代表暴露的端口 `connect` 代表监听的端口 配置好重启即可