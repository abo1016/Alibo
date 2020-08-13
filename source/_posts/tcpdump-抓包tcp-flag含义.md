---
title: tcpdump 抓包tcp flag含义
tags: []
id: '86'
categories:
  - - all-blog
    - 网络
date: 2018-05-25 17:12:13
---

#### TCP FLAG 标记

基于标记的TCP包匹配经常被用于过滤试图打开新连接的TCP数据包。

*   TCP标记和他们的意义如下所列：  
      
    \* F : FIN - 结束; 结束会话  
    \* S : SYN - 同步; 表示开始会话请求  
    \* R : RST - 复位;中断一个连接  
    \* P : PUSH - 推送; 数据包立即发送  
    \* A : ACK - 应答  
    \* U : URG - 紧急  
    \* E : ECE - 显式拥塞提醒回应  
    \* W : CWR - 拥塞窗口减少

示例：

*   三次握手Three-way Handshake

一个虚拟连接的建立是通过三次握手来实现的

1\. (B) --> \[SYN\] --> (A)

假如有服务器A、客户机B. 当B要和A通信时，B首先向A发一个SYN (Synchronize) 标记的包，告诉A请求建立连接.

注意: 一个 SYN包就是仅SYN标记设为1的TCP包(参见TCP包头Resources). 只有当A收到B发来的SYN包，才可建立连接，除此之外别无他法。

2\. (B) <-- \[SYN/ACK\] <--(A)

接着，A收到后会发一个对SYN包的确认包(SYN/ACK)回去，表示对第一个SYN包的确认，并继续握手操作.

注意: SYN/ACK包是仅SYN 和 ACK 标记为1的包.

3\. (B) --> \[ACK\] --> (A)

B收到SYN/ACK 包,B发一个确认包(ACK)，通知A连接已建立。至此，三次握手完成，一个TCP连接完成。

注意: ACK包就是仅ACK 标记设为1的TCP包.

特别注意：需要注意的是当三此握手完成、连接建立以后，TCP连接的每个包都会设置ACK位

PS：这就是为何连接跟踪很重要的原因了. 没有连接跟踪,防火墙将无法判断收到的ACK包是否属于一个已经建立的连接.一般的包过滤(Ipchains)收到ACK包时,会让它通过(这绝对不是个好主意). 而当状态型防火墙收到此种包时，它会先在连接表中查找是否属于哪个已建连接，否则丢弃该包

*   四次握手Four-way Handshake

四次握手用来关闭已建立的TCP连接

1\. (B) --> ACK/FIN --> (A)

2\. (B) <-- ACK <-- (A)

3\. (B) <-- ACK/FIN <-- (A)

4\. (B) --> ACK --> (A)

注意: 由于TCP连接是双向连接, 因此关闭连接需要在两个方向上做。ACK/FIN 包(ACK 和FIN 标记设为1)通常被认为是FIN(终结)包.然而, 由于连接还没有关闭, FIN包总是打上ACK标记. 没有ACK标记而仅有FIN标记的包不是合法的包，并且通常被认为是恶意的

*   连接复位Resetting a connection

四次握手不是关闭TCP连接的唯一方法. 有时,如果主机需要尽快关闭连接(或连接超时,端口或主机不可达),RST (Reset)包将被发送. 注意在，由于RST包不是TCP连接中的必须部分, 可以只发送RST包(即不带ACK标记). 但在正常的TCP连接中RST包可以带ACK确认标记。

注意：RST包是可以不要收到方确认的

*   无效的TCP标记Invalid TCP Flags

到目前为止，你已经看到了 SYN, ACK, FIN, 和RST 标记. 另外，还有PSH (Push) 和URG (Urgent)标记.

最常见的非法组合是SYN/FIN 包. 注意:由于 SYN包是用来初始化连接的, 它不可能和 FIN和RST标记一起出现. 这也是一个恶意攻击.

由于现在大多数防火墙已知 SYN/FIN 包, 别的一些组合,例如SYN/FIN/PSH, SYN/FIN/RST, SYN/FIN/RST/PSH。很明显，当网络中出现这种包时，很你的网络肯定受到攻击了。

别的已知的非法包有FIN (无ACK标记)和"NULL"包。如同早先讨论的，由于ACK/FIN包的出现是为了关闭一个TCP连接，那么正常的FIN包总是带有 ACK 标记。"NULL"包就是没有任何TCP标记的包(URG,ACK,PSH,RST,SYN,FIN都为0)。

到目前为止，正常的网络活动下，TCP协议栈不可能产生带有上面提到的任何一种标记组合的TCP包。当你发现这些不正常的包时，肯定有人对你的网络不怀好意。