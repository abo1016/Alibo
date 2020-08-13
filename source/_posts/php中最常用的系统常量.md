---
title: php中最常用的系统常量
tags: []
id: '315'
categories:
  - - all-blog
    - PHP
date: 2018-09-28 11:44:11
---

#### 系统常量

*   FILE 当前PHP文件的相对路径
    
*   LINE 当前PHP文件中所在的行号
    
*   FUNCTION 当前函数名，只对函数内调用起作用
    
*   CLASS 当前类名，只对类起作用
    
*   PHP\_VERSION 当前使用的PHP版本号
    
*   PHP\_OS 当前PHP环境的运行操作系统
    
*   TRUE 与true一样
    
*   FALSE 与false一样
    
*   M\_PI 圆周率常量值
    
*   M\_E 科学常数e
    
*   M\_LOG2E 代表log2
    
*   e，以2为底e的对数
    
*   M\_LOG10E 代表lg
    
*   e，以10为底e的对数
    
*   M\_LN2 2的自然对数
    
*   M\_LN10 10的自然对数
    
*   E\_ERROR 最近的错误之处
    
*   E\_WARNING 最近的警告之处
    
*   E\_PARSE 剖析语法有潜在问题之处
    
*   METHOD 表示类方法名，比如B::test
    

#### 服务器全局变量

*   $\_SERVER 返回服务器相关信息，返回一个数组
    
*   $\_GET 所有GET请求过来的参数
    
*   $\_POST 所有POST过来的参数
    
*   $\_COOKIE 所有HTTP提交过来的cookie
    
*   $\_FILES 所有HTTP提交过来的文件
    
*   $\_ENV 当前的执行环境信息
    
*   $\_REQUEST 相当于$\_POST、$\_GET、$\_COOKIE提交过来的数据，因此这个变量不值得信任
    
*   $\_SESSION session会话变量