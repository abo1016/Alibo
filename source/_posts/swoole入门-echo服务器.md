---
title: swoole入门-- Echo服务器
tags: [swoole]
id: '182'
categories:
  - PHP
date: 2018-06-22 09:27:34
---

#### 网络编程的 **“Hello World”**

> 创建一个server.php 作为服务器程序

```php
<?php
class Server
{
    private $serv;

    public function __construct() {
        $this->serv = new swoole_server("0.0.0.0", 9501);
        $this->serv->set(array(
            'worker_num' => 8,
            'daemonize' => false,
        ));

        $this->serv->on('Start', array($this, 'onStart'));
        $this->serv->on('Connect', array($this, 'onConnect'));
        $this->serv->on('Receive', array($this, 'onReceive'));
        $this->serv->on('Close', array($this, 'onClose'));

        $this->serv->start();
    }

    public function onStart( $serv ) {
        echo "Start\n";
    }

    public function onConnect( $serv, $fd, $from_id ) {
        $serv->send( $fd, "Hello {$fd}!" );
    }

    public function onReceive( swoole_server $serv, $fd, $from_id, $data ) {
        echo "Get Message From Client {$fd}:{$data}\n";
        $serv->send($fd, $data);
    }

    public function onClose( $serv, $fd, $from_id ) {
        echo "Client {$fd} close connection\n";
    }
}
// 启动服务器 Start the server
$server = new Server();
```

> 创建一个client.php 作为客户端程序

```php
<?php
class Client
{
    private $client;

    public function __construct() {
        $this->client = new swoole_client(SWOOLE_SOCK_TCP);
    }

    public function connect() {
        if( !$this->client->connect("127.0.0.1", 9501 , 1) ) {
            echo "Error: {$this->client->errMsg}[{$this->client->errCode}]\n";
        }

        fwrite(STDOUT, "请输入消息 Please input msg：");  
        $msg = trim(fgets(STDIN));
        $this->client->send( $msg );

        $message = $this->client->recv();
        echo "Get Message From Server:{$message}\n";
    }
}

$client = new Client();
$client->connect();
```

> 运行server.php

```
root@vagrant-ubuntu-trusty-64:~# php server.php 
[2018-06-22 01:18:00 @8674.0]   TRACE   Create swoole_server host=0.0.0.0, port=9501, mode=3, type=1
Start
```

> 运行client.php

```
root@vagrant-ubuntu-trusty-64:~# php client.php 

请输入消息 Please input msg：
```

> 通信

`请输入消息 Please input msg：hello php`

```
[2018-06-22 01:21:14 #8989.2]   TRACE   [Master] Accept new connection. maxfd=4|reactor_id=2|conn=25
[2018-06-22 01:21:14 *8994.1]   TRACE   [Worker] send: sendn=20|type=0|content=Hello 1!
[2018-06-22 01:22:21 #8989.1]   TRACE   send string package, size=9 bytes.
[2018-06-22 01:22:21 #8989.1]   TRACE   dispatch, type=10|len=9

Get Message From Client 1:hello php
```