---
title: php细节问题  int 0  和 空的区别判断
tags: []
id: '266'
categories:
  - - all-blog
    - PHP
date: 2018-07-26 11:52:56
---

### 对0的判断

```php
 $cast_id = 0;

var_dump(strlen($cast_id));   //1

var_dump(empty($cast_id)); // true

var_dump(isset($cast_id)); //true

var_dump(is_null($cast_id));//false
```

### 对空的判断

```php
 $cast_id = "";

var_dump(strlen($cast_id));   //0

var_dump(empty($cast_id)); // true

var_dump(isset($cast_id)); //true

var_dump(is_null($cast_id));//false

```

**通过php函数去区分0、null，'',的区别，可以直接判断strlen字节长度。**