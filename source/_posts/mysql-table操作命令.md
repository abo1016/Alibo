---
title: mysql table操作命令
tags: []
id: '106'
categories:
  - - all-blog
    - mysql
date: 2018-05-25 23:49:19
---

```
1、查看数据库中所有表
show tables；

2、创建表
create table user( id bigint not null auto_increment, name varchar(32) not null)charset = utf8,engine=innodb;

3、显示表结构
show columns from user;

4、显示表全部信息，包括表结构、表使用引擎、编码等
show create table user;

5、修改表编码
alter table user default character set gbk;

6、修改表引擎
alter table user engine = InnoDb;

7、新增列
alter table  user add age int not null;

8、删除列
alter table  user drop age;

9、修改列信息
alter table user modify age int not null default 18;
alter table user change age age int not null default 20 comment ‘年龄’;

ps:
ALTER TABLE 【表名字】 CHANGE 【列名称】【新列名称】 BIGINT NOT NULL  COMMENT ‘注释说明’

10、索引操作
查看索引:
show index from user;
10.1、普通索引
添加:ALTER TABLE 表名 ADD index索引名(列名) 或者 create index 索引名 on 表名(列名)
alter table user add index age1(age);
create index age1 on user(age)

删除:
alter table user drop index age1;
drop index age1 on user;

10.2、唯一索引
添加：ALTER TABLE 表名 ADD UNIQUE 索引名(列名) 或者 create UNIQUE 索引名 on 表名(列名)
alter table user add unique name(name);
create unique index name on user(name);
删除：
alter table user drop index name;

10.3、主键索引
添加:ALTER TABLE 表名 ADD primary key索引名(列名)
alter table user add primary key id(id);

修改为自增长属性：
alter table user change id id bigint(20) not null auto_increment;
alter table user change id id bigint(20) not null;

删除：
alter table user drop primary key

10.4、全局索引
只能适合在MyISAM引擎上

11、外键操作
（1）只有InnoDB类型的表才可以使用外键，mysql默认是MyISAM，这种类型不支持外键约束
（2）外键的好处：可以使得两张表关联，保证数据的一致性和实现一些级联操作；
（3）外键的作用：
保持数据一致性，完整性，主要目的是控制存储在外键表中的数据。 使两张表形成关联，外键只能引用外表中的列的值！
（4）建立外键的前提：
两个表必须是InnoDB表类型。
使用在外键关系的域必须为索引型(Index)。
使用在外键关系的域必须与数据类型相似
（5）创建的步骤
指定主键关键字： foreign key(列名)
引用外键关键字： references <外键表名>(外键列名)
（6）事件触发限制:on delete和on update , 可设参数cascade(跟随外键改动), restrict(限制外表中的外键改动),set Null(设空值）,set Default（设默认值）,[默认]no action

11.1、添加
语法:
alter table 表名 add constraint FK_ID(外键名) foreign key(外键字段名) REFERENCES 外表表名(对应的表的主键字段名);
例: 
alter table locstock add foreign key locstock_ibfk2(stockid) references product(stockid)
locstock 为表名, locstock_ibfk2 为外键名 第一个括号里填写外键列名, product为表名,第二个括号里是写外键关联的列名

12.2、删除：
ALTER   TABLE   表名   DROP   FOREIGN   KEY   外键名     //去掉外键唯一索引
1、查看数据库中所有表
show tables；

2、创建表
create table user( id bigint not null auto_increment, name varchar(32) not null)charset = utf8,engine=innodb;

3、显示表结构
show columns from user;

4、显示表全部信息，包括表结构、表使用引擎、编码等
show create table user;

5、修改表编码
alter table user default character set gbk;

6、修改表引擎
alter table user engine = InnoDb;

7、新增列
alter table  user add age int not null;

8、删除列
alter table  user drop age;

9、修改列信息
alter table user modify age int not null default 18;
alter table user change age age int not null default 20 comment ‘年龄’;

ps:
ALTER TABLE 【表名字】 CHANGE 【列名称】【新列名称】 BIGINT NOT NULL  COMMENT ‘注释说明’

10、索引操作
查看索引:
show index from user;
10.1、普通索引
添加:ALTER TABLE 表名 ADD index索引名(列名) 或者 create index 索引名 on 表名(列名)
alter table user add index age1(age);
create index age1 on user(age)

删除:
alter table user drop index age1;
drop index age1 on user;

10.2、唯一索引
添加：ALTER TABLE 表名 ADD UNIQUE 索引名(列名) 或者 create UNIQUE 索引名 on 表名(列名)
alter table user add unique name(name);
create unique index name on user(name);
删除：
alter table user drop index name;

10.3、主键索引
添加:ALTER TABLE 表名 ADD primary key索引名(列名)
alter table user add primary key id(id);

修改为自增长属性：
alter table user change id id bigint(20) not null auto_increment;
alter table user change id id bigint(20) not null;

删除：
alter table user drop primary key

10.4、全局索引
只能适合在MyISAM引擎上

11、外键操作
（1）只有InnoDB类型的表才可以使用外键，mysql默认是MyISAM，这种类型不支持外键约束
（2）外键的好处：可以使得两张表关联，保证数据的一致性和实现一些级联操作；
（3）外键的作用：
保持数据一致性，完整性，主要目的是控制存储在外键表中的数据。 使两张表形成关联，外键只能引用外表中的列的值！
（4）建立外键的前提：
两个表必须是InnoDB表类型。
使用在外键关系的域必须为索引型(Index)。
使用在外键关系的域必须与数据类型相似
（5）创建的步骤
指定主键关键字： foreign key(列名)
引用外键关键字： references <外键表名>(外键列名)
（6）事件触发限制:on delete和on update , 可设参数cascade(跟随外键改动), restrict(限制外表中的外键改动),set Null(设空值）,set Default（设默认值）,[默认]no action

11.1、添加
语法:
alter table 表名 add constraint FK_ID(外键名) foreign key(外键字段名) REFERENCES 外表表名(对应的表的主键字段名);
例: 
alter table locstock add foreign key locstock_ibfk2(stockid) references product(stockid)
locstock 为表名, locstock_ibfk2 为外键名 第一个括号里填写外键列名, product为表名,第二个括号里是写外键关联的列名

12.2、删除：
ALTER   TABLE   表名   DROP   FOREIGN   KEY   外键名     //去掉外键唯一索引
```