---
title: Golang --新的玩具
tags: []
id: '140'
categories:
  - - all-blog
    - Golang
date: 2018-06-20 16:45:43
---

### Golang 简介

![logo](https://upload-images.jianshu.io/upload_images/3462294-882a2d694e522197.jpg) Golang是Google公司在2009年发布的一款开源编程语言，在2012年早些时候发布了Go 1.0稳定版本。现在Go的开发已经是完全开放的，并且拥有一个活跃的社区。背靠Google爸爸，Golang在互联网领域发展飞快，在国内外大厂（Google、Facebook、alibaba、tencent、jd 等等）流行程度大有拳打java脚踢C++的趋势。在大数据和分布式平台领域Golang可以说是一个合格的贡献者，特别是后续的人工智能领域的发展潜力巨大。平时工作中的主力语言是脚本语言，所以想通过学习一门编译语言来补充脚本语言在某些情况的缺陷并跟进时代的脚步，这是我学习Golang的初衷。

#### Golang特色

*   比PHP、Python等解释性语言快，但语法简洁程度不亚于它们。
*   并发性能某些场景比肩java
*   部署简易，安全可靠
*   C/C++能做的后台工作Golang都可以胜任，在开发效率上可以秒杀前者，因为是编译语言，性能方面也不会有非常大的差距。

* * *

#### 安装

安装Go工具 **Linux、Mac OS X 和 FreeBSD 的安装包** 下载此压缩包并提取到 /usr/local 目录，在 /usr/local/go 中创建Go目录树。例如： `tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz` 该压缩包的名称可能不同，这取决于你安装的Go版本和你的操作系统以及处理器架构。 （此命令必须作为root或通过 sudo 运行。） 要将 /usr/local/go/bin 添加到 PATH 环境变量， 你需要将此行添加到你的 /etc/profile（全系统安装）或 $HOME/.profile 文件中： `export PATH=$PATH:/usr/local/go/bin` **安装到指定位置** Go二进制发行版假定它们会被安装到 /usr/local/go （或Windows下的 c:\\Go）中，但也可将Go工具安装到不同的位置。 此时你必须设置 GOROOT 环境变量来指出它所安装的位置。 例如，若你将Go安装到你的home目录下，你应当将以下命令添加到 $HOME/.profile 文件中：

```
export GOROOT=$HOME/go
export PATH=$PATH:$GOROOT/bin
```

**注：GOROOT 仅在安装到指定位置时才需要设置。** 对于Windows用户，Go项目提供两种安装选项（从源码安装除外）： zip压缩包需要你设置一些环境变量，而实验性MSI安装程序则会自动配置你的安装。

#### 测试安装是否成功

##### 第一个Go语言程序 --- HelloGolang.go

```go
package main
import "fmt"

func main(){
    fmt.Printf("hello,golang\n")
}
```

_运行：_

```
go run HelloGolang.go

hello,golang
```