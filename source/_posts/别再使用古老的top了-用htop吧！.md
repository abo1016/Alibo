---
title: 别再使用古老的top了  用htop吧！
tags: [top, Htop]
id: '219'
categories:
    - linux
date: 2018-07-11 17:34:35
---

> 在linux系统中我们经常会使用top命令去查看进程状况，系统资源占用状况。

[![Pu1MNT.md.png](https://s1.ax1x.com/2018/07/11/Pu1MNT.md.png)](https://imgchr.com/i/Pu1MNT)

> **现在我们有了更好的选择** -- htop

htop 是Linux系统中的一个互动的进程查看器，一个文本模式的应用程序(在控制台或者X终端中)，需要ncurses。与Linux传统的top相比，htop更加人性化。它可让用户交互式操作，支持颜色主题，可横向或纵向滚动浏览进程列表，并支持鼠标操作。 与top相比，htop有以下优点：

*   可以横向或纵向滚动浏览进程列表，以便看到所有的进程和完整的命令行。
*   在启动上，比top 更快。
*   杀进程时不需要输入进程号。
*   htop 支持鼠标操作。

**要是htop第一步先安装吧** `apt-get install htop` #htop命令打开 `root@vagrant-ubuntu-trusty-64:/www/# htop` [![Pu1z24.md.png](https://s1.ax1x.com/2018/07/11/Pu1z24.md.png)](https://imgchr.com/i/Pu1z24) htop整体布局和top很相似，上面左上角显示CPU、内存、交换区的使用情况，右边显示任务、负载、开机时间，下面就是进程实时状况。并且可以以进程树的方式来显示，在设置可以设置默认显示。 下面是 F1~F10 的功能和对应的字母快捷键。

Shortcut Key | Function Key | Description                   | 中文说明                    
------------ | ------------ | ----------------------------- | ------------------------
h, ?         | F1           | Invoke htop Help              | 查看htop使用说明              
S            | F2           | Htop Setup Menu               | htop 设定                 
/            | F3           | Search for a Process          | 搜索进程                    
\            | F4           | Incremental process filtering | 增量进程过滤器                 
T            | F5           | Tree View                     | 显示树形结构                  
<,>          | F6           | Sort by a column              | 选择排序方式                  
[            | F7           | Nice – (change priority)      | 可减少nice值，这样就可以提高对应进程的优先级
]            | F8           | Nice – (change priority)      | 可增加nice值，这样就可以降低对应进程的优先级
k            | F9           | Kill a Process                | 可对进程传递信号                
q            | F10          | Quit htop                     | 结束htop                  

**命令行选项（COMMAND-LINE OPTIONS）** -C --no-color　　　　 　　 使用一个单色的配色方案 -d --delay=DELAY　　　　 设置延迟更新时间，单位秒 -h --help　　　　　　 　　 显示htop 命令帮助信息 -u --user=USERNAME　　 只显示一个给定的用户的过程 -p --pid=PID,PID…　　　 只显示给定的PIDs -s --sort-key COLUMN　 依此列来排序 -v –version　　　　　　　 显示版本信息 **交互式命令（INTERACTIVE COMMANDS）** 上下键或**PgUP**, **PgDn** 选定想要的进程，左右键或**Home**, **End** 移动字段，当然也可以直接用鼠标选定进程； **Space** 标记/取消标记一个进程。命令可以作用于多个进程，例如 **"kill"**，将应用于所有已标记的进程 **U** 取消标记所有进程 **s** 选择某一进程，按s:用strace追踪进程的系统调用 **l** 显示进程打开的文件: 如果安装了lsof，按此键可以显示进程所打开的文件 **I** 倒转排序顺序，如果排序是正序的，则反转成倒序的，反之亦然 **+, -** When in tree view mode, expand or collapse subtree. When a subtree is collapsed a "+" sign shows to the left of the process name. **a** (在有多处理器的机器上) 设置 CPU affinity: 标记一个进程允许使用哪些CPU **u** 显示特定用户进程 **M** 按Memory 使用排序 **P** 按CPU 使用排序 **T** 按Time+ 使用排序 **F** 跟踪进程: 如果排序顺序引起选定的进程在列表上到处移动，让选定条跟随该进程。这对监视一个进程非常有用：通过这种方式，你可以让一个进程在屏幕上一直可见。使用方向键会停止该功能。 **K** 显示/隐藏内核线程 **H** 显示/隐藏用户线程 **Ctrl-L** 刷新 **Numbers** PID 查找: 输入PID，光标将移动到相应的进程上
<!-- more -->
