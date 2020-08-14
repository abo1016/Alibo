---
title: mysql 用户授权
tags: [mysql, 权限]
id: '210'
categories:
    - mysql
date: 2018-07-02 14:47:18
---

> 新建用户

```sql
CREATE USER username IDENTIFIED BY 'password';
```

> 分配用户权限

```sql
GRANT ALL PRIVILEGES ON *.* TO 'username'@'localhost' IDENTIFIED BY 'password';
```

> 撤销用户权限 重新分配

```sql
EVOKE ALL PRIVILEGES ON *.* FROM 'username'@'localhost';
GRANT ALL PRIVILEGES ON wordpress.* TO 'username'@'localhost' IDENTIFIED BY 'password';
```

> 单一权限授权 如下，授权select update权限

```sql
GRANT SELECT, UPDATE ON wordpress.* TO 'username'@'localhost' IDENTIFIED BY 'password';

```

> 刷新权限

```sql
FLUSH PRIVILEGES;
```

> 删除用户

```sql
DROP USER username@localhost;
```

> 查看用户

```sql
SELECT User, Host FROM user;
```

> 查看用户授权状态

```sql
 show grants for user;
```