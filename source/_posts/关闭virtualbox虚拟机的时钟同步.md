---
title: 关闭VirtualBox虚拟机的时钟同步
tags: []
id: '298'
categories:
  - - all-blog
    - vagrant
date: 2018-09-05 17:16:23
---

**使用VirtualBox作为虚拟操作系统载体，有时候我们需要主机与虚拟机时间同步一致，有时候需要两者之间时间不一致，经过整理主要存在以下两种方案。**

#### 方案1

*   关闭时间同步

```shell
VBoxManage setextradata <虚拟机名/虚拟机UUID> "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled" "1"
```

*   打开时间同步

```shell
VBoxManage setextradata <虚拟机名/虚拟机UUID> "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled" "0"
```

#### 方案2

*   关闭时间同步 `vboxmanage guestproperty set <虚拟机名/虚拟机UUID> --timesync-set-stop`
    
*   打开时间同步 `vboxmanage guestproperty set <虚拟机名/虚拟机UUID> --timesync-set-start`