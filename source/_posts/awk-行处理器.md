---
title: awk 行处理器
tags: []
id: '216'
categories:
  - - all-blog
    - linux
    - shell
date: 2018-07-10 16:45:17
---

> **linux 强大的awk命令** 相比较屏幕处理的优点，在处理庞大文件时不会出现内存溢出或是处理缓慢的问题，通常用来格式化文本信息

#### 命令行

`awk [-F|-f|-v] 'BEGIN{} //{command1; command2} END{}' file` `[-F|-f|-v]` 大参数，-F指定分隔符，-f调用脚本，-v定义变量 var=value `''` 引用代码块 `BEGIN` 初始代码块，在对每一行进行处理之前，初始化代码，引用全局变量，设置分割符FS `//` 匹配代码块，可以使字符串或正则表达式 `{}` 命令代码块，包含一条或多条命令 `;` 多条命令使用分号分隔 `END` 结尾代码块，在对每一行进行处理之后再执行的代码块，主要是进行最终计算或输出结尾摘要信息

#### 要点

`$0` 表示当前行 `$1` 每行一个字段 `NF` 字段数量 `NR` 行号 `FNR` 与NR类似，不过多文件记录不递增，每个文件都从1开始 `\t、\n` 格式化符号 `FS` BEGIN时定义分隔符 `RS` 输入的记录分隔符， 默认为换行符(即文本是按一行一行输入) `~` 匹配，与==相比不是精确比较 `!~` 不匹配，不精确比较 `==` 等于，必须全部相等，精确比较 `!=` 不等于，精确比较 `&&` 逻辑与 `||` 逻辑或 `+` 匹配时表示1个或1个以上 `/[0-9][0-9]+/` 两个或两个以上数字 `/[0-9][0-9]*/` 一个或一个以上数字 `FILENAME` 文件名 `OFS` 输出字段分隔符， 默认也是空格，可以改为制表符等 `ORS` 输出的记录分隔符，默认为换行符,即处理结果也是一行一行输出到屏幕 `-F'[:#/]'` 定义三个分隔符 **print** print 是awk打印指定内容的主要命令 `awk '{print}' /etc/passwd == awk '{print $0}' /etc/passwd` `awk '{print " "}' /etc/passwd` 每处理一行输出一个空行 `awk '{print "a"}' /etc/passwd` 每处理一行输出一个行且行内容只有a字符 `awk -F":" '{print $1}' /etc/passwd` 以：为分隔符每行输出一个字段 `awk -F: '{print $1; print $2}' /etc/passwd` 以：为分隔符分行输出第一个字段和第二个字段 `awk -F: '{print $1,$3,$6}' OFS="\t" /etc/passwd` 输出字段1,3,6，以制表符作为分隔符 **\-f指定脚本文件** `awk -f script.awk file`

```shell
BEGIN{
FS=":"
}
{print $1}
```

效果与awk -F":" '{print $1}'相同,只是分隔符使用FS在代码自身中指定 `ls -l|awk 'BEGIN{sum=0} !/^d/{sum+=$5} END{print "total size is",sum}'` 计算当前目录文件总大小 **\-F指定分隔符** `awk -F'[:#/]' '{print NF}' test.sh` 指定三个分隔符，并输出每行字段数 **//匹配代码块 //纯字符匹配 !//纯字符不匹配 ~//字段值匹配 !~//字段值不匹配 ~/a1|a2/字段值匹配a1或a2** `awk '/mysql/' /etc/passwd` 拼配mysql字符 `awk '!/mysql/{print $0}' /etc/passwd` 输出不匹配mysql的行 `awk -F: '/mail/,/mysql/{print}' /etc/passwd` 区间匹配 `awk -F: '$1~/mail/{print $1}' /etc/passwd` $1匹配指定内容才显示 `awk -F: '{if($1~/mail/) print $1}' /etc/passwd` 与上面相同 `awk -F: '$1!~/mail/{print $1}' /etc/passwd` 不匹配 **if语句** `awk -F: '{if($1~/mail/) print $1}' /etc/passwd == awk -F: '{if($1~/mail/) {print $1}}' /etc/passwd` `awk -F: '{if($1~/mail/) {print $1} else {print $2}}' /etc/passwd` **条件表达式 == != > >=**

```shell
awk -F":" '$1=="mysql"{print $3}' /etc/passwd  
awk -F":" '{if($1=="mysql") print $3}' /etc/passwd          //与上面相同 
awk -F":" '$1!="mysql"{print $3}' /etc/passwd                 //不等于
awk -F":" '$3>1000{print $3}' /etc/passwd                      //大于
awk -F":" '$3>=100{print $3}' /etc/passwd                     //大于等于
awk -F":" '$3<1{print $3}' /etc/passwd                            //小于
awk -F":" '$3<=1{print $3}' /etc/passwd
```

\*\*逻辑运算符 &&　|| \*\*

```shell
awk -F: '$1~/mail/ && $3>8 {print }' /etc/passwd         //逻辑与，$1匹配mail，并且$3>8
awk -F: '{if($1~/mail/ && $3>8) print }' /etc/passwd
awk -F: '$1~/mail/ || $3>1000 {print }' /etc/passwd       //逻辑或
awk -F: '{if($1~/mail/ || $3>1000) print }' /etc/passwd   //取整
```

**输出分隔符OFS** `awk -F ":" '{print $1,$3}' OFS="\n" /etc/passwd`

```shell
root
0
daemon
1
bin
2
sys
3
sync
4
games
5
man
6
lp
7
mail
8
news
9
uucp

```

**while语句**

```shell
awk -F: 'BEGIN{i=1} {while(i<NF) print NF,$i,i++}' /etc/passwd 
7 root 1
7 x 2
7 0 3
7 0 4
7 root 5
7 /root 6
```

**数组**

```shell
netstat -anp|awk 'NR!=1{a[$6]++} END{for (i in a) print i,"\t",a[i]}'
netstat -anp|awk 'NR!=1{a[$6]++} END{for (i in a) printf "%-20s %-10s %-5s \n", i,"\t",a[i]}'
9523                               1
9929                               1
LISTEN                            6
7903                               1
3038/cupsd                          1
10837                             1
9833                               1
```