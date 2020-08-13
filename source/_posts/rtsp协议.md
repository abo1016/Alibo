---
title: RTSP协议
tags: []
id: '436'
categories:
  - - all-blog
    - 网络
date: 2019-05-13 21:53:14
---

最近公司有个需求，对接alexa 平台的camera skill。就是将ipc的视频流放到智能带屏音箱上播放出来，这个需求看似好像比较简单其实坑的一匹:sob: 首先我们具备的基础条件是：

*   公司的ipc没有公网网卡，视频流只能在局域网中播放。
*   现有ipc是为server模式，并且只接受被动请求，不会主动推rtsp流。
*   Amazon alexa 平台硬性要求所有的rtsp流需要进行亚马逊认证的CA TLS: ![](https://blog.wenboo.top/wp-content/uploads/2019/05/eb6f877c69981e9482f4a7ae50d72880.png)

**面临的问题：** 1. 局域网的rtsp流无法进行CA认证的TLS加密 2. 就算搭建拉砖推流服务器再加密，但是局域网的ip行拉取。

#### 解决方案：

1.  搭建rtsp流转发服务器，公司现有ipc不具备推流条件（舍弃）
2.  在alexa skill store 有看到一家解决方案 **Monocle**

> ![](https://blog.wenboo.top/wp-content/uploads/2019/05/3b7c79a670f8fe4fffd295ee1833a13f.png)

他们通过认证网关的形式来达到目的，但是具体的实现不知。下一步得先学习下RTSP协议的内容 [RFC 2326](https://tools.ietf.org/html/rfc2326 "RFC 2326")。

> 从RTSP协议文档中我们可以看到，RTSP具备和HTTP类似的重定向功能(redirect 3xx)

\`A redirect request informs the client that it must connect to another server location. It contains the mandatory header Location, which indicates that the client should issue requests for that URL. It may contain the parameter Range, which indicates when the redirection takes effect. If the client wants to continue to send or receive media for this URI, the client MUST issue a TEARDOWN request for the current session and a SETUP for the new session at the designated host. This example request redirects traffic for this URI to the new server at the given play time:

```
 S->C: REDIRECT rtsp://example.com/fizzle/foo RTSP/1.0
       CSeq: 732
       Location: rtsp://bigserver.com:8001
       Range: clock=19960213T143205Z-`
```

**我猜想Monocle的实现方法应该就是利用了这一个特性** 开始去网上找轮子，github ，Google，百度搜了个遍基本没有现成的轮子。这个方法用的人应该非常少，著名的RTSP开源项目Live555也没有实现这个标准特性。 所以我又开始学习Live555的源码（从没写过C++的，表示这真的是在难为我啊~~~)想通过修改源码来实现这个功能。 ![](https://blog.wenboo.top/wp-content/uploads/2019/05/d4a9763c42ee32ebfe4b6d44bd036a2c.png) 不过目前离我的猜想已经很近了，现在通过抓包可以清晰的看到了RTSP协议的交互过程。 C表示RTSP客户端,S表示RTSP服务端

1.  **第一步：查询服务器端可用方法**

1.C->S:OPTION request //询问S有哪些方法可用 1.S->C:OPTION response //S回应信息的public头字段中包括提供的所有可用方法

2.  **第二步：得到媒体描述信息**

2.C->S:DESCRIBE request //要求得到S提供的媒体描述信息 2.S->C:DESCRIBE response //S回应媒体描述信息，一般是sdp信息

3.  **第三步：建立\*\*\*\*RTSP\*\*\*\*会话**

3.C->S:SETUP request //通过Transport头字段列出可接受的传输选项，请求S建立会话 3.S->C:SETUP response //S建立会话，通过Transport头字段返回选择的具体转输选项，并返回建立的Session ID;

4.  **第四步：请求开始传送数据**

4.C->S:PLAY request //C请求S开始发送数据 4.S->C:PLAY response //S回应该请求的信息

5.  **第五步：** \*\*\*\*\*\*数据传送播放中\*\*

S->C:发送流媒体数据 // 通过RTP协议传送数据

6.  **第六步：关闭会话，退出**

6.C->S:TEARDOWN request //C请求关闭会话 6.S->C:TEARDOWN response //S回应该请求 ![](https://blog.wenboo.top/wp-content/uploads/2019/05/f73b01c1e1d5a8fac285228863196c78.png) ![](https://blog.wenboo.top/wp-content/uploads/2019/05/49f4629698a5327a89a2d973207bd66d.png) 可以修改通过修改SETUP Phase当中的交互返回来实现。