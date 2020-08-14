---
title: select、poll、epoll
tags: [多路复用模型]
id: '460'
categories:
    - linux
date: 2019-06-14 11:48:05
---

**select、poll、epoll 模型，都是为多路io复用模型，它们有哪些不同和优缺点：**

### 阻塞 io 模型 blocking IO

最常用的也就是阻塞io模型。默认情况下，所有文件操作都是阻塞的。我们以套接字接口为例来讲解此模型，在进程空间调用recvfrom，其系统调用知道数据包到达并且被复制到进程缓冲中或者发生错误时才会返回，在此期间会一直阻塞，所以进程在调用recvfrom开始到它返回的整段时间都是阻塞的，因此称之为阻塞io模型。 ![](https://blog.wenboo.top/wp-content/uploads/2019/06/4d051d24389fb4ce866b70efba564a98.png)

### 非阻塞 io 模型 nonblocking IO

应用层数据到kernel的过程中，recvfrom会轮询检查，如果kernel数据没有准备还，就返回一个EWOULDBLOCK错误。不断的轮询检查，直到发现kernel中的数据准备好了，就返回，然后进行系统调用，将数据从kernel拷贝到进程缓冲区中。类似busy-waiting的方法 ![](https://blog.wenboo.top/wp-content/uploads/2019/06/b8be846cdb8aed3745782e47054e8570.png)

### I/O 多路复用（ IO multiplexing）

使用一个进程来处理多个io fd，资源利用率高。select,poll,epoll模型通过轮询的方式来发现io请求。

*   select

原理：当前进程在调用select function时整个进程进入**阻塞**，此时内核开始监控select 所处理的这个socket io，当中任意socket连接有数据，则会立即返回。用户socket进程进入read状态，内核将数据复制到用户进程缓冲区。 ![](https://blog.wenboo.top/wp-content/uploads/2019/06/e2a8b3d25cf0f7bbe25ab4fbde60b320.png) I/O 多路复用的特点是通过一种机制一个进程能同时等待多个文件描述符，而这些文件描述符（套接字描述符）其中的任意一个进入读就绪状态，select()函数就可以返回。 缺点：

1.  每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
    
2.  同时每次调用select都需要在内核遍历传递进来的所有fd，这个开销在fd很多时也很大
    
3.  select支持的文件描述符数量太小了，默认是1024
    

### poll

与select模型基本相似。不同之处 1. 在于poll对于fd并没有最大数量限制（但是数量过大后性能也是会下降

2.  poll有一个特点是水平触发，也就是通知程序fd就绪后，这次没有被处理，那么下次poll的时候会再次通知同个fd已经就绪

### epoll

**增强版的select和poll** 1. epoll同样和poll没有fd的数量限制

2.  它处理方式式通过回调函数的方式，每一个请求fd都会注册事件回调，触发回调函数将fd和事件状态返回给poll对象。
    
3.  从2可以看出epoll与select，poll的主动轮询区别，epoll是事件驱动被动触发方式，对于大量fd且不是所有都活跃的情况下性能更加好。