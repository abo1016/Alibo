---
title: PHP 的几种异步执行方式
tags: [异步]
id: '275'
categories:
    - PHP
date: 2018-08-07 13:39:51
---

**有时候一些任务的执行用户根本不关心执行结果，但是需要用户等待任务执行完成后才能关闭客户端，在php单线程的机制下，任务执行到一部分被关闭的情况非常不友好。**

### ajax

在前端编程中就知道ajax可以进行异步执行。

```php
$.get("doAsync.php", { name: 'raykaeso',job:'PHP Programmer'} );
```

### 使用popen，执行本地文件

```php
pclose(popen('php /var/www/doAsync.php &', 'r'));
```

### 使用curl超时

设置curl的超时时间 CURLOPT\_TIMEOUT 为1 （最小为1），因此客户端需要等待1秒，curl请求地址必须为绝对路径

```php
$param = array(
'name'=>'raykaeso',
'job'=>'PHP Programmer'
);
$ch = curl_init();
curl_setopt($ch, CURLOPT_URL,"http://www.example.com/doAsync.php");
curl_setopt($ch, CURLOPT_POST, 1);
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($param)); //将数组转换为URL请求字符串
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
curl_setopt($ch, CURLOPT_HEADER, false);
curl_setopt($ch, CURLOPT_TIMEOUT, 1);
curl_exec($ch);
curl_close($ch);
```

### 使用shell\_exec命令

```php
shell_exec("/use/local/php/bin/php /www/test.php  > /dev/null 2>&1 &");
```

shell中可能经常能看到：>/dev/null 2>&1 命令的结果可以通过%>的形式来定义输出 /dev/null 代表空设备文件 不阻塞可以将命令输出的内容写入系统的一个回收站文件，这样程序就不会阻塞 此种方法可创建独立Apache或nginx等HTTP服务之外的php进程，不影响HTTP接收其他请求。 缺点：在并发处理时会创建大量的php进程