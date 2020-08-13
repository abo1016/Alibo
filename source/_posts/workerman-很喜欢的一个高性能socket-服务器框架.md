---
title: workerman --很喜欢的一个高性能socket 服务器框架
tags: []
id: '134'
categories:
  - - all-blog
    - PHP
date: 2018-06-08 09:54:51
---

# 最近项目中一直在使用wokerman 确切的说应该是GatewayWorker（基于wokerman封装的TCP长连接应用框架）。

GatewayWorker是基于Workerman开发的一套**TCP长连接**的应用框架，实现了单发、群发、广播等接口，内置了mysql类库，GatewayWorker分为Gateway进程和Worker进程，天然支持分布式部署，能够支持庞大的连接数（百万甚至千万连接级别的应用）。  
可用于开发IM聊天应用、移动通讯、游戏后台、物联网、智能家居后台等等。GatewayWorker是不支持udp的，若需要udp服务请选择wokerman。  

  

GatewayWorker使用经典的Gateway和Worker进程模型。Gateway进程负责维持客户端连接，并转发客户端的数据给BusinessWorker进程处理，BusinessWorker进程负责处理实际的业务逻辑（默认调用Events.php处理业务），并将结果推送给对应的客户端。Gateway服务和BusinessWorker服务可以分开部署在不同的服务器上，实现分布式集群。  

GatewayWorker提供非常方便的API，可以全局广播数据、可以向某个群体广播数据、也可以向某个特定客户端推送数据。配合Workerman的定时器，也可以定时推送数据。  

# GatewayWorker特性

### 1、基于Workerman开发

GatewayWorker是基于Workerman开发的

### 2、基于Gateway、Worker进程模型

GatewayWorker使用经典的Gateway和Worker进程模型。Gateway进程负责维持客户端连接，并转发客户端的数据给Worker进程处理；Worker进程负责处理实际的业务逻辑，并将结果推送给对应的客户端。Gateway服务和Worker服务可以分开部署在不同的服务器上，实现分布式集群。

### 3、支持分布式部署

GatewayWorker可以非常方便实现分布式部署，Gateway服务和Worker服务都可以分开部署在不同的服务器集群上。并且操作简单、容易扩容、上下线用户无感知。

### 4、支持高并发

Gateway进程只负责网络IO，Worker进程负责业务逻辑。其中每个Gateway进程可以维持上万的并发连接，多个Gateway进程可以维持数十万甚至百万的并发连接，Gateway集群则可以维持千万级别的并发连接。

### 5、支持全局广播或者向任意客户端推送数据

GatewayWorker提供非常方便的API，可以全局广播数据、可以向某个群体广播数据、也可以向某个特定客户端推送数据。配合Workerman的定时器，也可以定时推送数据。

### 6、支持各种应用层协议

WorkerMan接口上支持各种应用层协议，包括自定义协议。同样GatewayWorker也支持各种应用层协议。

### 7、多协议支持

有时应用客户端所使用的协议不止一种，例如PC网页客户端使用的是WebSocket协议，而手机App使用的是其它协议。GatewayWorker可以非常方便的支持多协议，只需要以不同的协议开不同的端口即可，业务代码无需改动。

### 8、支持对象或者资源永久保持

WorkerMan在运行过程中只会载入解析一次PHP文件，然后便常驻内存，这使得类及函数声明、PHP执行环境、符号表等不会重复创建销毁，这与Web容器下运行的PHP机制是完全不同的。在WorkerMan中，一个进程生命周期内静态成员或者全局变量在不主动销毁的情况下是永久保持的，也就是将对象或者链接等资源放到全局变量或者类静态成员中则整个进程生命周期内的所有请求都可以复用。例如只要单个进程内初始化一次数据库连接，则以后这个进程的所有请求都可以复用这个数据库连接，避免了频繁连接数据库过程中TCP三次握手、 数据库权限验证、断开连接时TCP四次握手的过程，极大的提高了应用程序效率。

### 9、高性能

由于php文件从磁盘读取解析一次后便会常驻内存，下次使用时直接使用内存中的opcode， 极大的减少了磁盘IO及PHP中请求初始化、创建执行环境、词法解析、语法解析、编译opcode、请求关闭等诸多耗时过程， 并且不依赖nginx、apache等容器，少了nginx等容器与PHP通信的开销，最主要的是资源可以永久保持，不必每次初始化数据库连接等等， 所以使用WorkerMan开发应用程序，性能非常高。

### 10、支持HHVM

支持在HHVM虚拟机上运行，可成倍提升PHP性能。尤其是在cpu密集运算业务中，性能非常优异，是PHP Zend虚拟机8倍左右。通过实际压力测试对比，在没有负载业务的情况下，WorkerMan在HHVM下运行比在Zend PHP5.6运行网络吞吐量提高了30-80%左右

### 11、方便与其它项目集成

针对其它项目，GatewayWorker提供推送非常简单方便的API，可以在任何项目中使用这个API向所有客户端或者特定客户端推送数据，比如在普通Web项目中推送数据。

### 12、支持代码热更新

可以reload Worker进程实现业务代码更新升级，而不必担心客户端连接会断开，因为客户端连接都由Gateway进程维持。

### 13、支持长连接

GatewayWorker主要用于长连接即时通讯应用。如游戏服务器、物联网云服务、IM、移动应用等。

  

workerman的手册对开发者非常友好，基本每一个点都写的很清晰明了，有兴趣的同学可以去官网看看它的手册。

手册地址：https://doc2.workerman.net/ 

  

下面讲下最近昨天用的很多的定时器（纯PHP的定时器，简直不要太高级啊！！！）

  

Timer::add(float $time\_interval, callable $callback \[,$args = array(), bool $persistent = true\]) //创建一个定时器

注意：定时器是在当前进程中运行的，workerman中不会创建新的进程或者线程去运行定时器。

### 参数

time\_interval

多长时间执行一次，单位秒，支持小数，可以精确到0.001，即精确到毫秒级别。

callback

回调函数注意：如果回调函数是类的方法，则方法必须是public属性

args

回调函数的参数，必须为数组，数组元素为参数值

persistent

是否是持久的，如果只想定时执行一次，则传递false（只执行一次的任务在执行完毕后会自动销毁，不必调用Timer::del()）。默认是true，即一直定时执行。

每个定时器都会返回一个 timerid，我们可以通过Timer::del($timerid)销毁这个定时器。

**用法：**

  

**1\. 定时函数为匿名函数（闭包）**

// 每2.5秒执行一次 

  $time\_interval = 2.5;
    Timer::add($time\_interval, function() { echo "task run\\n";
    });

  

  

**2****.** **定时函数为匿名函数，利用闭包传递参数**  

  

  

// 每10秒执行一次 $time\_interval = 10;
    $connect\_time = time(); // 给connection对象临时添加一个timer\_id属性保存定时器id $connection->timer\_id = Timer::add($time\_interval, function()use($connect\_time) {
         $connection->send($connect\_time);
    });

	

### 
	3、定时器函数为匿名函数，利用定时器接口传递参数

// 每10秒执行一次 $time\_interval = 10;
    $connect\_time = time(); // 给connection对象临时添加一个timer\_id属性保存定时器id $connection->timer\_id = Timer::add($time\_interval, function($connection, $connect\_time) {
         $connection->send($connect\_time);
    }, array($connection, $connect\_time));
}; // 连接关闭时，删除对应连接的定时器 $ws\_worker->onClose = function($connection) { // 删除定时器 Timer::del($connection->timer\_id);
};

### 
	4、定时函数为普通函数

	  

function send\_mail($to, $content){}

// 10秒后执行发送邮件任务，最后一个参数传递false，表示只运行一次 Timer::add(10, 'send\_mail', array($to, $content), false);

  

### 
	5、定时函数为类的方法

	  

class Mail { // 注意，回调函数属性必须是public public function send($to, $content) { echo "send mail ...\\n";

   }

}

// 10秒后发送一次邮件 $mail = new Mail();
    $to = 'workerman@workerman.net';
    $content = 'hello workerman';
    Timer::add(10, array($mail, 'send'), array($to, $content), false);

### 
	6、定时函数为类的静态方法

	  

class Mail { 

// 注意这个是静态方法，回调函数属性也必须是public public static function send($to, $content) { echo "send mail ...\\n";
    }
}

// 10秒后发送一次邮件 $to = 'workerman@workerman.net';
    $content = 'hello workerman'; // 定时调用类的静态方法 Timer::add(10, array('Mail', 'send'), array($to, $content), false);