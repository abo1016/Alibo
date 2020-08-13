---
title: workerman压力测试
tags: []
id: '213'
categories:
  - - all-blog
    - PHP
date: 2018-07-10 11:29:47
---

> **公司香港服务器wokerman 出现内存泄露问题** 怀疑是否是框架自身问题

#### 压力测试脚本

```php
require_once './Workerman/Autoloader.php';
use Workerman\Worker;
$worker = new Worker('tcp://0.0.0.0:1234');
// 进程数配置成cpu核数-1，保留一个cpu给ab进程
$worker->count= 1;
$worker->onMessage = function($connection, $data)
{
    $connection->send("HTTP/1.1 200 OK\r\nConnection: keep-alive\r\nServer: workerman\1.1.4\r\n\r\nhello");
};
Worker::runAll();
```

#### 使用Apache自带工具ab 来模拟请求 安装 apt-get install apache2-utils

```shell
ab -n1000000 -c100 -k http://127.0.0.1:1234/
```

> **结果**

```shell
root@vagrant-ubuntu-trusty-64:/www/cloud/mj_ipc# ab -n1000000 -c100 -k http://127.0.0.1:1234/
This is ApacheBench, Version 2.3 <$Revision: 1528965 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient)
Completed 100000 requests
Completed 200000 requests
Completed 300000 requests
Completed 400000 requests
Completed 500000 requests
Completed 600000 requests
Completed 700000 requests
Completed 800000 requests
Completed 900000 requests
Completed 1000000 requests
Finished 1000000 requests


Server Software:        workerman
Server Hostname:        127.0.0.1
Server Port:            1234

Document Path:          /
Document Length:        5 bytes

Concurrency Level:      100
Time taken for tests:   16.332 seconds
Complete requests:      1000000
Failed requests:        0
Keep-Alive requests:    1000000
Total transferred:      72000000 bytes
HTML transferred:       5000000 bytes
Requests per second:    61227.94 [#/sec] (mean)
Time per request:       1.633 [ms] (mean)
Time per request:       0.016 [ms] (mean, across all concurrent requests)
Transfer rate:          4305.09 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.0      0       6
Processing:     0    2   0.3      2      10
Waiting:        0    2   0.3      2      10
Total:          0    2   0.3      2      17

Percentage of the requests served within a certain time (ms)
  50%      2
  66%      2
  75%      2
  80%      2
  90%      2
  95%      2
  98%      2
  99%      3
 100%     17 (longest request)
```

> ps：本压测脚本业务逻辑简单 可根据情况自行修改