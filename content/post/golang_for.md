+++
author = "jayzhou"
title = "Golang进阶--深入理解Go语言中的for循环与闭包陷阱"
date = "2025-02-07"
description = "golang语言注意点--for循环里的闭包"
featured = true
tags = [
    "golang",
    "进阶",
    "for循环"
]
categories = [
    "golang",
    "闭包",
]
series = ["golang","golang进阶"]
aliases = ["闭包"]
thumbnail = "images/golang_closure.png"
+++

本文主要讨论了golang里面闭包的一个可能遇到的坑。
<!--more-->
golang在使用for循环的时候，尤其是在结合goroutine和闭包时，可能会遇到一些意想不到的问题。本文将通过一个具体的例子，深入探讨这些问题及其解决方案。
## 问题描述
以下是一个简单的Go程序，它使用for循环遍历一个切片，并在每次迭代中启动一个goroutine来打印切片中的元素及其地址：
{{< highlight golang >}}
package main

import (
    "fmt"
    "time"
)

func main() {
    arr := make([]int, 0)
    arr = append(arr, 10)
    arr = append(arr, 20)
    arr = append(arr, 30)
    arr = append(arr, 40)

    for _, a := range arr {
        fmt.Printf("arr addr:%p, num:%d\n", &a, a)
        go func() {
            fmt.Printf("addr:%p, num:%d\n", &a, a)
        }()
    }

    time.Sleep(time.Second * 5)
}
{{< /highlight >}}
从输出中可以看到，for循环中的变量a的地址始终是相同的，而goroutine中打印的值并不总是与预期一致。

## 问题分析
### 1.变量a的地址问题
在for循环中，变量a是循环变量的一个实例。每次迭代时，a的值会被更新为切片中的下一个元素，但其地址并不会改变。这是因为a是一个局部变量，它的地址在每次迭代中都是相同的。
### 2.闭包中的变量捕获
在goroutine中使用的闭包捕获了变量a。由于goroutine是并发执行的，它们可能会在for循环结束后才开始执行。此时，a的值已经被更新为切片中的最后一个元素（即40），因此所有goroutine都会打印出40。
### 3.并发执行的速度问题
由于goroutine的执行速度可能赶不上for循环的迭代速度，因此在goroutine开始执行时，a的值可能已经被更新为切片中的某个中间值（如30或20），从而导致输出结果不一致。
## 解决方案
为了避免上述问题，可以在每次迭代时将a的值传递给goroutine，而不是直接捕获a。这样可以确保每个goroutine都有自己的变量副本，从而避免竞争条件。

以下是修改后的代码：
{{< highlight golang >}}
package main

import (
    "fmt"
    "time"
)

func main() {
    arr := make([]int, 0)
    arr = append(arr, 10)
    arr = append(arr, 20)
    arr = append(arr, 30)
    arr = append(arr, 40)

    for index, a := range arr {
        fmt.Printf("arr addr:%p, num:%d\n", &a, a)
        go func(i, index int) {
            fmt.Printf("arr %d: addr:%p, num:%d\n", index, &i, i)
        }(a, index)
    }

    time.Sleep(time.Second * 5)
}
{{< /highlight >}}
在这个修改后的版本中，goroutine接收两个参数：i和index。i是a的副本，index是当前迭代的索引。由于每个goroutine都有自己的i和index副本，因此不会出现竞争条件。
## 总结
在Go语言中，for循环与goroutine结合使用时，需要注意闭包中的变量捕获问题。为了避免竞争条件，应该将循环变量作为参数传递给goroutine，而不是直接捕获循环变量。这样可以确保每个goroutine都有自己的变量副本，从而避免意外的行为。
