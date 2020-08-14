---
title: PHP的过滤器函数
tags: [php, 过滤]
id: '303'
categories:
    - PHP
date: 2018-09-13 17:55:49
---

**PHP中很屌但经常被各种忽略的过滤器函数** PHP其实自带一些功能强大的函数，在我们实现某个逻辑或功能时，经常不会用PHP自带的好用的函数，而大多时候都是选择绕远路的方法自己去实现。

#### 先拉两个出来遛遛

**1\. filter\_has\_var** 判断$\_GET中是否有某个字段参数通常的写法

```php
if(isset($_GET[“name”])  //或者post参数、或者cookie里
```

其实还可以这么写

```php
filter_has_var(INPUT_GET, ‘name’) //可以直接返回true或false  第一个参数 可以填 INPUT_GET、 INPUT_POST、 INPUT_COOKIE、 INPUT_SERVER、 INPUT_ENV 看英文你应该知道 是干啥的
```

**2\. filter\_var()** 我们勤劳的phper通常来验证邮件或者电话的格式时，都会辛辛苦苦的通过正则来实现。 来看下PHP可以的工具吧

```php
filter_var(‘shenyi@hishenyi.com, FILTER_VALIDATE_EMAIL);
```

对，就这么简单~~~~ 以前的代码白写了。 还有这么多，真是太方便了。。 [![iA57Wt.md.png](https://s1.ax1x.com/2018/09/13/iA57Wt.md.png)](https://imgchr.com/i/iA57Wt) **还有更屌的净化过滤** FILTER\_SANITIZE\_NUMBER\_INT 过滤掉非数字型的内容

```php
echo filter_var(‘abc123’, FILTER_SANITIZE_NUMBER_INT);   //直接返回123
```

还有更多。。。 [![iAIQl6.md.png](https://s1.ax1x.com/2018/09/13/iAIQl6.md.png)](https://imgchr.com/i/iAIQl6) 不得不说PHP真的是世界上最好的语言啊 哈哈哈哈哈哈 [![滑稽](https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=1926707095,4273239503&fm=27&gp=0.jpg "滑稽")](https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=1926707095,4273239503&fm=27&gp=0.jpg "滑稽")