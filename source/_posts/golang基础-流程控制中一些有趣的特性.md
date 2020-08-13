---
title: golang基础  流程控制中一些有趣的特性
tags: []
id: '347'
categories:
  - - all-blog
    - Golang
date: 2018-12-04 16:08:57
---

#### 最近在看golang基础，在流程控制这部分看到了一些golang特有的好用的又有点高大上特性，记录一下。

> 多个返回值

直接看代码

```go
package main

import "fmt"

//返回 A+B 和 A*B
func SumAndProduct(A, B int) (int, int) {
    return A+B, A*B
}

func main() {
    x := 3
    y := 4

    xPLUSy, xTIMESy := SumAndProduct(x, y)

    fmt.Printf("%d + %d = %d\n", x, y, xPLUSy)
    fmt.Printf("%d * %d = %d\n", x, y, xTIMESy)
}
```

上面的例子我们可以看到直接返回了两个参数，当然我们也可以命名返回参数的变量，这个例子里面只是用了两个类型，我们也可以改成如下这样的定义，然后返回的时候不用带上变量名，因为直接在函数里面初始化了。但如果你的函数是导出的(首字母大写)，官方建议：最好命名返回值，因为不命名返回值，虽然使得代码更加简洁了，但是会造成生成的文档可读性差。

```go
func SumAndProduct(A, B int) (add int, Multiplied int) {
    add = A+B
    Multiplied = A*B
    return
}
```

> 变参

Go函数支持变参。接受变参的函数是有着不定数量的参数的。为了做到这点，首先需要定义函数使其接受变参：

```go
func myfunc(arg ...int) {}
```

arg ...int告诉Go这个函数接受不定数量的参数。注意，这些参数的类型全部是int。在函数体中，变量arg是一个int的slice：

```go
for _, n := range arg {
    fmt.Printf("And the number is: %d\n", n)
}
```

> defer

Go语言中有种不错的设计，即延迟（defer）语句，你可以在函数中添加多个defer语句。当函数执行到最后时，这些defer语句会按照逆序执行，最后该函数返回。特别是当你在进行一些打开资源的操作时，遇到错误需要提前返回，在返回前你需要关闭相应的资源，不然很容易造成资源泄露等问题。如下代码所示，我们一般写打开一个资源是这样操作的：

```go
func ReadWrite() bool {
    file.Open("file")
// 做一些工作
    if failureX {
        file.Close()
        return false
    }

    if failureY {
        file.Close()
        return false
    }

    file.Close()
    return true
}
```

我们看到上面有很多重复的代码，Go的defer有效解决了这个问题。使用它后，不但代码量减少了很多，而且程序变得更优雅。在defer后指定的函数会在函数退出前调用。

```go
func ReadWrite() bool {
    file.Open("file")
    defer file.Close()
    if failureX {
        return false
    }
    if failureY {
        return false
    }
    return true
}
```

如果有很多调用defer，那么defer是采用后进先出模式，所以如下代码会输出4 3 2 1 0

```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```