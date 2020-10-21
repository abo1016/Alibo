---
title: HTTP/2
tags: [HTTP2]
id: '405'
categories:
    - 网络
date: 2019-03-06 10:53:35
---

**\#1 HTTP2**

![架构师谈基于HTTP2推送消息到APNs](http://p1.pstatp.com/large/pgc-image/15254174738146085162958)

HTTP1.0最早在网页中使用是在1996年，那个时候只是使用一些较为简单的网页上和网络请求上，而HTTP1.1则在1999年才开始广泛应用于现在的各大浏览器网络请求中，同时HTTP1.1也是当前使用最为广泛的HTTP协议。

在 HTTP/1.0 的时候，client每请求一项资源，都必须先建立一次 TCP 连线，而在 client 收到 server 的 response后，便会断开TCP连线。而在 HTTP/1.1 的时代，允许同域名下的资源 request and response 后，才断开 TCP 连线。

![架构师谈基于HTTP2推送消息到APNs](http://p1.pstatp.com/large/pgc-image/1525417473947b700cac46d)

HTTP2 是 HTTP/1.1 后的一次重大的改进，在协议层面改善了以上问题，减少资源占用，来，直接感受一下差异：

HTTP/2 is the future of the Web, and it is here!

这是 Akamai 公司建立的一个官方的演示，用以说明 HTTP/2 相比于之前的 HTTP/1.1 在性能上的大幅度提升。 同时请求 379 张图片，从Load time 的对比可以看出 HTTP/2 在速度上的优势。

![架构师谈基于HTTP2推送消息到APNs](http://p9.pstatp.com/large/pgc-image/1525417473899180e242782)

HTTP/2 源自 SPDY/2。SPDY 系列协议由谷歌开发，于 2009 年公开。它的设计目标是降低 50% 的页面加载时间。当下很多著名的互联网公司都在自己的网站或 APP 中采用了 SPDY 系列协议（当前最新版本是 SPDY/3.1），因为它对性能的提升是显而易见的。主流的浏览器（谷歌、火狐、Opera）也都早已经支持 SPDY，它已经成为了工业标准，HTTP Working-Group 最终决定以 SPDY/2 为基础，开发 HTTP/2。HTTP/2 标准于2015年5月以 RFC 7540 正式发表。

HTTP/2 的5个特色：

**Binary Framing Layer**

HTTP/2 采用二进制格式传输数据，而非 HTTP/1.x 的文本格式。

![架构师谈基于HTTP2推送消息到APNs](http://p1.pstatp.com/large/pgc-image/152541747386539f2fabf67)

在 HTTP/1.X 由 OSI model 中的 Application Layer，存在着 Binary Framing Layer，记录着 HTTP 的内容像 HEADER 中的 method、content、message 等内容，以及 Data 部分。

![架构师谈基于HTTP2推送消息到APNs](http://p3.pstatp.com/large/pgc-image/1525417473882912fa06f4f)

Frame 的基本格式如下：

![架构师谈基于HTTP2推送消息到APNs](http://p9.pstatp.com/large/pgc-image/1525417473846a70f8c5365)

一个基础的 Stream 由 HEADER frame 与 Data frame 组成。(共有10种 frame)

而每次的 connection 可以乘载着任意数量的 stream。

![架构师谈基于HTTP2推送消息到APNs](http://p1.pstatp.com/large/pgc-image/152541747387724cdd3737f)

可以用 chrome 内部自带的工具（chrome://net-internals/）查看 HTTP2 流量，但这个包信息量比较少，结构不如我们熟悉的 Fiddler or Wireshark 清晰。

用 wireshark 抓包：

![架构师谈基于HTTP2推送消息到APNs](http://p3.pstatp.com/large/pgc-image/152541747392846d2dc37aa)

一个包内有多个不同的 Steam ID

![架构师谈基于HTTP2推送消息到APNs](http://p1.pstatp.com/large/pgc-image/15254174738973d65032c4c)

**Multiplexing**

Multiplexing 允许单一的 tcp 有多重请求/回应，也就是说 client 和 server 可以将 http 请求/回应分解成不藕合的 frame，然后随机发送，最后在另一端根据 stream ID 将 freame 组合起来。

![架构师谈基于HTTP2推送消息到APNs](http://p1.pstatp.com/large/pgc-image/152541747385933b6e160fc)

**Request Prioritization**

在 HTTP/2 中，stream 有著 priority 的属性，而藉由 Priorty frame，便可以建立起 priority tree。

![架构师谈基于HTTP2推送消息到APNs](http://p3.pstatp.com/large/pgc-image/1525417473905cafa2f0b76)

**Header Compression**

在 client 和 server 个维护一个 HPACK，採用 hash 的方式来记录 HEADER 的内容。也就是说当 client 要请求资源前会先去 HPACK 查找缺失的资源，接著请求缺少的资源。

![架构师谈基于HTTP2推送消息到APNs](http://p1.pstatp.com/large/pgc-image/1525417473887ce96d4e76b)

**Server Push**

在 Binary Framing Layer 提过，frame 共有10种。在这裡会利用 PUSH_PROMISE 的frame，它用于 server 主动发送资源给 client。

![架构师谈基于HTTP2推送消息到APNs](http://p1.pstatp.com/large/pgc-image/1525417473894999f15dbf0)