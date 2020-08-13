---
title: php 高并发处理
tags: []
id: '400'
categories:
  - - all-blog
    - PHP
date: 2019-02-26 10:44:43
---

转：[https://www.cnblogs.com/walblog/articles/8476579.html](https://www.cnblogs.com/walblog/articles/8476579.html "https://www.cnblogs.com/walblog/articles/8476579.html") 我们通常衡量一个Web系统的吞吐率的指标是QPS（Query Per Second，每秒处理请求数），解决每秒数万次的高并发场景，这个指标非常关键。举个例子，我们假设处理一个业务请求平均响应时间为100ms，同时，系统内有20台Apache的Web服务器，配置MaxClients为500个（表示Apache的最大连接数目）。 那么，我们的Web系统的理论峰值QPS为（理想化的计算方式）： 20\*500/0.1 = 100000 （10万QPS） 咦？我们的系统似乎很强大，1秒钟可以处理完10万的请求，5w/s的秒杀似乎是“纸老虎”哈。实际情况，当然没有这么理想。在高并发的实际场景下，机器都处于高负载的状态，在这个时候平均响应时间会被大大增加。 普通的一个p4的服务器每天最多能支持大约10万左右的IP，如果访问量超过10W那么需要专用的服务器才能解决，如果硬件不给力 软件怎么优化都是于事无补的。主要影响服务器的速度 有：网络-硬盘读写速度-内存大小-cpu处理速度。 就Web服务器而言，Apache打开了越多的连接进程，CPU需要处理的上下文切换也越多，额外增加了CPU的消耗，然后就直接导致平均响应时间增加。因此上述的MaxClient数目，要根据CPU、内存等硬件因素综合考虑，绝对不是越多越好。可以通过Apache自带的abench来测试一下，取一个合适的值。然后，我们选择内存操作级别的存储的Redis，在高并发的状态下，存储的响应时间至关重要。网络带宽虽然也是一个因素，不过，这种请求数据包一般比较小，一般很少成为请求的瓶颈。负载均衡成为系统瓶颈的情况比较少，在这里不做讨论哈。 那么问题来了，假设我们的系统，在5w/s的高并发状态下，平均响应时间从100ms变为250ms（实际情况，甚至更多）： 20\*500/0.25 = 40000 （4万QPS） 于是，我们的系统剩下了4w的QPS，面对5w每秒的请求，中间相差了1w。 举个例子，高速路口，1秒钟来5部车，每秒通过5部车，高速路口运作正常。突然，这个路口1秒钟只能通过4部车，车流量仍然依旧，结果必定出现大塞车。（5条车道忽然变成4条车道的感觉） 同理，某一个秒内，20\*500个可用连接进程都在满负荷工作中，却仍然有1万个新来请求，没有连接进程可用，系统陷入到异常状态也是预期之内。 ![14834077821.jpg](http://img.php.cn//upload/image/146/269/839/1485234211796818.jpg "1485234211796818.jpg") 其实在正常的非高并发的业务场景中，也有类似的情况出现，某个业务请求接口出现问题，响应时间极慢，将整个Web请求响应时间拉得很长，逐渐将Web服务器的可用连接数占满，其他正常的业务请求，无连接进程可用。 更可怕的问题是，是用户的行为特点，系统越是不可用，用户的点击越频繁，恶性循环最终导致“雪崩”（其中一台Web机器挂了，导致流量分散到其他正常工作的机器上，再导致正常的机器也挂，然后恶性循环），将整个Web系统拖垮。

3.  重启与过载保护

如果系统发生“雪崩”，贸然重启服务，是无法解决问题的。最常见的现象是，启动起来后，立刻挂掉。这个时候，最好在入口层将流量拒绝，然后再将重启。如果是redis/memcache这种服务也挂了，重启的时候需要注意“预热”，并且很可能需要比较长的时间。 秒杀和抢购的场景，流量往往是超乎我们系统的准备和想象的。这个时候，过载保护是必要的。如果检测到系统满负载状态，拒绝请求也是一种保护措施。在前端设置过滤是最简单的方式，但是，这种做法是被用户“千夫所指”的行为。更合适一点的是，将过载保护设置在CGI入口层，快速将客户的直接请求返回 高并发下的数据安全 我们知道在多线程写入同一个文件的时候，会存现“线程安全”的问题（多个线程同时运行同一段代码，如果每次运行结果和单线程运行的结果是一样的，结果和预期相同，就是线程安全的）。如果是MySQL数据库，可以使用它自带的锁机制很好的解决问题，但是，在大规模并发的场景中，是不推荐使用MySQL的。秒杀和抢购的场景中，还有另外一个问题，就是“超发”，如果在这方面控制不慎，会产生发送过多的情况。我们也曾经听说过，某些电商搞抢购活动，买家成功拍下后，商家却不承认订单有效，拒绝发货。这里的问题，也许并不一定是商家奸诈，而是系统技术层面存在超发风险导致的。

1.  超发的原因

假设某个抢购场景中，我们一共只有100个商品，在最后一刻，我们已经消耗了99个商品，仅剩最后一个。这个时候，系统发来多个并发请求，这批请求读取到的商品余量都是99个，然后都通过了这一个余量判断，最终导致超发。（同文章前面说的场景） ![14834077822.jpg](http://img.php.cn//upload/image/398/176/480/1485234279277458.jpg "1485234279277458.jpg") 在上面的这个图中，就导致了并发用户B也“抢购成功”，多让一个人获得了商品。这种场景，在高并发的情况下非常容易出现。 优化方案1：将库存字段number字段设为unsigned，当库存为0时，因为字段不能为负数，将会返回false

```php
<?php
//优化方案1：将库存字段number字段设为unsigned，当库存为0时，因为字段不能为负数，将会返回false
include('./mysql.php');
$username = 'wang'.rand(0,1000);
//生成唯一订单
function build_order_no(){
  return date('ymd').substr(implode(NULL, array_map('ord', str_split(substr(uniqid(), 7, 13), 1))), 0, 8);
}
//记录日志
function insertLog($event,$type=0,$username){
    global $conn;
    $sql="insert into ih_log(event,type,usernma)
    values('$event','$type','$username')";
    return mysqli_query($conn,$sql);
}
function insertOrder($order_sn,$user_id,$goods_id,$sku_id,$price,$username,$number)
{
      global $conn;
      $sql="insert into ih_order(order_sn,user_id,goods_id,sku_id,price,username,number)
      values('$order_sn','$user_id','$goods_id','$sku_id','$price','$username','$number')";
     return  mysqli_query($conn,$sql);
}
//模拟下单操作
//库存是否大于0
$sql="select number from ih_store where goods_id='$goods_id' and sku_id='$sku_id' ";
$rs=mysqli_query($conn,$sql);
$row = $rs->fetch_assoc();
  if($row['number']>0){//高并发下会导致超卖
      if($row['number']<$number){
        return insertLog('库存不够',3,$username);
      }
      $order_sn=build_order_no();
      //库存减少
      $sql="update ih_store set number=number-{$number} where sku_id='$sku_id' and number>0";
      $store_rs=mysqli_query($conn,$sql);
      if($store_rs){
          //生成订单
          insertOrder($order_sn,$user_id,$goods_id,$sku_id,$price,$username,$number);
          insertLog('库存减少成功',1,$username);
      }else{
          insertLog('库存减少失败',2,$username);
      }
  }else{
      insertLog('库存不够',3,$username);
  }
?>
```

2.  悲观锁思路

解决线程安全的思路很多，可以从“悲观锁”的方向开始讨论。 悲观锁，也就是在修改数据的时候，采用锁定状态，排斥外部请求的修改。遇到加锁的状态，就必须等待。 ![14834077833.jpg](http://img.php.cn//upload/image/386/682/712/1485234326781590.jpg "1485234326781590.jpg") 虽然上述的方案的确解决了线程安全的问题，但是，别忘记，我们的场景是“高并发”。也就是说，会很多这样的修改请求，每个请求都需要等待“锁”，某些线程可能永远都没有机会抢到这个“锁”，这种请求就会死在那里。同时，这种请求会很多，瞬间增大系统的平均响应时间，结果是可用连接数被耗尽，系统陷入异常。 优化方案2：使用MySQL的事务，锁住操作的行

```php
<?php
//优化方案2：使用MySQL的事务，锁住操作的行
include('./mysql.php');
//生成唯一订单号
function build_order_no(){
  return date('ymd').substr(implode(NULL, array_map('ord', str_split(substr(uniqid(), 7, 13), 1))), 0, 8);
}
//记录日志
function insertLog($event,$type=0){
    global $conn;
    $sql="insert into ih_log(event,type)
    values('$event','$type')";
    mysqli_query($conn,$sql);
}
//模拟下单操作
//库存是否大于0
mysqli_query($conn,"BEGIN");  //开始事务
$sql="select number from ih_store where goods_id='$goods_id' and sku_id='$sku_id' FOR UPDATE";//此时这条记录被锁住,其它事务必须等待此次事务提交后才能执行
$rs=mysqli_query($conn,$sql);
$row=$rs->fetch_assoc();
if($row['number']>0){
    //生成订单
    $order_sn=build_order_no();
    $sql="insert into ih_order(order_sn,user_id,goods_id,sku_id,price)
    values('$order_sn','$user_id','$goods_id','$sku_id','$price')";
    $order_rs=mysqli_query($conn,$sql);
    //库存减少
    $sql="update ih_store set number=number-{$number} where sku_id='$sku_id'";
    $store_rs=mysqli_query($conn,$sql);
    if($store_rs){
      echo '库存减少成功';
        insertLog('库存减少成功');
        mysqli_query($conn,"COMMIT");//事务提交即解锁
    }else{
      echo '库存减少失败';
        insertLog('库存减少失败');
    }
}else{
  echo '库存不够';
    insertLog('库存不够');
    mysqli_query($conn,"ROLLBACK");
}
?>
```

3.  FIFO队列思路

那好，那么我们稍微修改一下上面的场景，我们直接将请求放入队列中的，采用FIFO（First Input First Output，先进先出），这样的话，我们就不会导致某些请求永远获取不到锁。看到这里，是不是有点强行将多线程变成单线程的感觉哈。 ![14834077834.jpg](http://img.php.cn//upload/image/744/620/541/1485234417677669.jpg "1485234417677669.jpg") 然后，我们现在解决了锁的问题，全部请求采用“先进先出”的队列方式来处理。那么新的问题来了，高并发的场景下，因为请求很多，很可能一瞬间将队列内存“撑爆”，然后系统又陷入到了异常状态。或者设计一个极大的内存队列，也是一种方案，但是，系统处理完一个队列内请求的速度根本无法和疯狂涌入队列中的数目相比。也就是说，队列内的请求会越积累越多，最终Web系统平均响应时候还是会大幅下降，系统还是陷入异常。

4.  文件锁的思路

对于日IP不高或者说并发数不是很大的应用，一般不用考虑这些！用一般的文件操作方法完全没有问题。但如果并发高，在我们对文件进行读写操作时，很有可能多个进程对进一文件进行操作，如果这时不对文件的访问进行相应的独占，就容易造成数据丢失 优化方案4：使用非阻塞的文件排他锁

```php
<?php
//优化方案4：使用非阻塞的文件排他锁
include ('./mysql.php');
//生成唯一订单号
function build_order_no(){
  return date('ymd').substr(implode(NULL, array_map('ord', str_split(substr(uniqid(), 7, 13), 1))), 0, 8);
}
//记录日志
function insertLog($event,$type=0){
    global $conn;
    $sql="insert into ih_log(event,type)
    values('$event','$type')";
    mysqli_query($conn,$sql);
}
$fp = fopen("lock.txt", "w+");
if(!flock($fp,LOCK_EX | LOCK_NB)){
    echo "系统繁忙，请稍后再试";
    return;
}
//下单
$sql="select number from ih_store where goods_id='$goods_id' and sku_id='$sku_id'";
$rs =  mysqli_query($conn,$sql);
$row = $rs->fetch_assoc();
if($row['number']>0){//库存是否大于0
    //模拟下单操作
    $order_sn=build_order_no();
    $sql="insert into ih_order(order_sn,user_id,goods_id,sku_id,price)
    values('$order_sn','$user_id','$goods_id','$sku_id','$price')";
    $order_rs =  mysqli_query($conn,$sql);
    //库存减少
    $sql="update ih_store set number=number-{$number} where sku_id='$sku_id'";
    $store_rs =  mysqli_query($conn,$sql);
    if($store_rs){
      echo '库存减少成功';
        insertLog('库存减少成功');
        flock($fp,LOCK_UN);//释放锁
    }else{
      echo '库存减少失败';
        insertLog('库存减少失败');
    }
}else{
  echo '库存不够';
    insertLog('库存不够');
}
fclose($fp);
 ?>
```

5.  乐观锁思路

这个时候，我们就可以讨论一下“乐观锁”的思路了。乐观锁，是相对于“悲观锁”采用更为宽松的加锁机制，大都是采用带版本号（Version）更新。实现就是，这个数据所有请求都有资格去修改，但会获得一个该数据的版本号，只有版本号符合的才能更新成功，其他的返回抢购失败。这样的话，我们就不需要考虑队列的问题，不过，它会增大CPU的计算开销。但是，综合来说，这是一个比较好的解决方案。 ![14834077835.jpg](http://img.php.cn//upload/image/542/338/822/1485234456400515.jpg "1485234456400515.jpg") 有很多软件和服务都“乐观锁”功能的支持，例如Redis中的watch就是其中之一。通过这个实现，我们保证了数据的安全。 优化方案5：Redis中的watch

```php
<?php
$redis = new redis();
 $result = $redis->connect('127.0.0.1', 6379);
 echo $mywatchkey = $redis->get("mywatchkey");
/*
  //插入抢购数据
 if($mywatchkey>0)
 {
     $redis->watch("mywatchkey");
  //启动一个新的事务。
    $redis->multi();
   $redis->set("mywatchkey",$mywatchkey-1);
   $result = $redis->exec();
   if($result) {
      $redis->hSet("watchkeylist","user_".mt_rand(1,99999),time());
      $watchkeylist = $redis->hGetAll("watchkeylist");
        echo "抢购成功！<br/>";
        $re = $mywatchkey - 1;  
        echo "剩余数量：".$re."<br/>";
        echo "用户列表：<pre>";
        print_r($watchkeylist);
   }else{
      echo "手气不好，再抢购！";exit;
   } 
 }else{
     // $redis->hSet("watchkeylist","user_".mt_rand(1,99999),"12");
     //  $watchkeylist = $redis->hGetAll("watchkeylist");
        echo "fail！<br/>";   
        echo ".no result<br/>";
        echo "用户列表：<pre>";
      //  var_dump($watchkeylist); 
 }*/
$rob_total = 100;   //抢购数量
if($mywatchkey<=$rob_total){
    $redis->watch("mywatchkey");
    $redis->multi(); //在当前连接上启动一个新的事务。
    //插入抢购数据
    $redis->set("mywatchkey",$mywatchkey+1);
    $rob_result = $redis->exec();
    if($rob_result){
         $redis->hSet("watchkeylist","user_".mt_rand(1, 9999),$mywatchkey);
        $mywatchlist = $redis->hGetAll("watchkeylist");
        echo "抢购成功！<br/>";

        echo "剩余数量：".($rob_total-$mywatchkey-1)."<br/>";
        echo "用户列表：<pre>";
        var_dump($mywatchlist);
    }else{
          $redis->hSet("watchkeylist","user_".mt_rand(1, 9999),'meiqiangdao');
        echo "手气不好，再抢购！";exit;
    }
}
?>
```