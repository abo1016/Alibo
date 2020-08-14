---
title: php socket C/S IO阻塞方式
tags: [php socket, socket,I/O]
id: '390'
categories:
    - PHP
date: 2019-02-16 13:59:54
---

#### 网络编程阻塞模式

**在网网络编程中通常有以下几种调用方式：**

*   同步调用（Sync）
*   异步调用（Async）
*   阻塞(Block)
*   非阻塞(Unblock)

**这几种调用方式可以进行组合：**

*   同步阻塞
*   同步非阻塞
*   异步阻塞
*   异步非阻塞

我们在通过php进行网络编程时会发现，php的会有IO阻塞的现象。在多个客户端同时连接服务器端时，服务器端会以客户端连接的先后顺序来处理客户端发送的请求，形成一个顺序队列的方式。当服务端在处理一个客户端连接时，其他的客户端是与服务器进行交互的，只有等待前一个连接处理结束。 **server/client php代码示例** 服务端

```php
<?php
set_time_limit(0);//确保连接客户端不会超时

$ip = '127.0.0.1';
$port = 4321;

//创建socket
if(($socket = socket_create(AF_INET, STREAM_SOCK_STREAM, SOL_TCP)) < 0){
    echo "socket_create() 失败的原因是:".socket_strerror(socket_last_error())."\n";
}

//阻塞模式
socket_set_block($socket) or die("socket_set_block() 失败的原因是:" . socket_strerror(socket_last_error()) . "\n");

//绑定ip和端口
if(($ret = socket_bind($socket, $ip, $port)) < 0){
    echo "socket_bind() 失败的原因是:".socket_strerror($ret)."\n";
}

//监听
if(($ret = socket_listen($socket)) < 0){
    echo "socket_listen() 失败的原因是:".socket_strerror($ret)."\n";
}
echo "PHP Socket Server create success!\n";
$buf = null;
do{
    // 接受一个Socket客户端连接
    if(($client = socket_accept($socket)) < 0){
        echo "socket_accept() failed: reason: " . socket_strerror(socket_last_error($client)) . "\n";
    }else{
        socket_getpeername($client, $client_addr, $client_port);
        //告知客户端连接成功
        $msg = "connect success!";
        socket_write($client, $msg, strlen($msg));
        do{
            //打印客户端发送来的信息
            if(($buf = @socket_read($client, 8192)) === false){
                echo "socket_read() failed: reason: " . socket_strerror(socket_last_error($client)) . "\n";
                socket_close($client);
                break;
            }
            echo "rev client[{$client_addr}]:".$buf."\n";
            if('exit' == $buf){
                break;
            }
            //服务器发送给客户端
            $buf = "server send: ($buf)";
            socket_write($client, $buf, strlen($buf));

        }while(true);
    }
}while(true);
//关闭socket
socket_close($socket);
```

客户端

```php
<?php

error_reporting(E_ALL);
set_time_limit(0);

echo "start connect server\n";

$ip = '127.0.0.1';
$port = 4321;

//创建socket
if(($socket = socket_create(AF_INET, STREAM_SOCK_STREAM, SOL_TCP)) < 0){
    echo "socket_create() failed: reason: " . socket_strerror(socket_last_error()) . "\n";
}

/***设置socket连接选项，这两个步骤可以省略***/
//接收套接流的最大超时时间1秒，后面是微秒单位超时时间，设置为零，表示不管它
socket_set_option($socket, SOL_SOCKET, SO_RCVTIMEO, array("sec" => 1, "usec" => 0));
//发送套接流的最大超时时间为6秒
socket_set_option($socket, SOL_SOCKET, SO_SNDTIMEO, array("sec" => 6, "usec" => 0));
/***设置socket连接选项，这两个步骤可以省略***/

$info = "connect $ip:$port...\n";
echo $info;

//连接到127.0.0.1:4321这个socket server上面
if(($ret = socket_connect($socket, $ip, $port)) < 0){
    echo "socket_connect() failed.\nReason: ($ret) " . socket_strerror($ret) . "\n";
}

do{
    //读取服务器发送信息
    $out = socket_read($socket, 8192);
    echo "client 接收:",$out."\n";

    //读取键盘输入并发送给服务器
    $in = trim(fgets(STDIN));

    if(! socket_write($socket, $in, strlen($in))){
        echo "socket_write() failed: reason: " . socket_strerror(socket_last_error($socket)) . "\n";
    }

    //判断是否退出命令
    if('exit' == $in){
        echo "client quit\n";
        break;
    }

}while(true);
//关闭socket
socket_close($socket);
```

运行server： ![](https://blog.wenboo.top/wp-content/uploads/2019/02/9765cc61c56df054f7d81d678964e3b3.png) 运行client 1 ![](https://blog.wenboo.top/wp-content/uploads/2019/02/bd6dcdf1a6fd7eb42b012f0489f82ff0.png) 服务端响应 ![](https://blog.wenboo.top/wp-content/uploads/2019/02/b37117313704f3a854f249f73b089c0f.png) 保持Client 1不断开 运行Client 2 程序 ![](https://blog.wenboo.top/wp-content/uploads/2019/02/11d7577a91fdfb2ebc04d1be10362779.png) 服务端没有任何响应，说明此时Client2客户端是阻塞状态 断开Client1后，服务端接收到了Client2的数据 ![](https://blog.wenboo.top/wp-content/uploads/2019/02/017634ef64e7701619bafa4a0a872001.png)