+++
author = "jayzhou"
title = "Golang基础--一篇文章记住操作符的所有用法"
date = "2024-02-27"
description = "golang基础介绍第三篇，介绍golang里面的操作符计算规则以及结果"
featured = true
tags = [
    "golang",
    "golang基础",
    "操作符"
]
categories = [
    "golang",
    "操作符",
]
series = ["golang","golang基础"]
aliases = ["golang操作符"]
thumbnail = "images/operator.png"
+++

本文比较简单，主要介绍在golang里面操作符(加减乘除以及取模)的几个要注意的点。
<!--more-->

## 速记
1. 如果除法两个操作数都是整数，则结果也是整数。
2. %取模运算符的结果没有float类型，正负号只与第一个操作数一致。
3. %取模运算操作数有类型限制。
3. 除此之外的运算符合并无特殊。

## 详细分析
### 运算结果
1. 加减乘结果都与实际人为计算一致。
2. 除法运算符有点特殊，如果除法两个操作数都是整数，则结果也是整数。比如5/4 结果并不是1.25，而是1，这与我们人为计算是不一致的。
2. %取模运算符的结果没有float类型，正负号只与第一个操作数一致，5%4,5%-4都是正数，-5%4，-5%-4都是负数。

### 操作数类型限制(适用于加减乘除运算符)
1. %取模运算仅能用于整数，无论这两个操作数是不是变量还是常量。
2. 如果两个操作数都是常量，则没有任何要求。
3. 如果两个操作数都是变量，则这两个操作数类型必须一致。
4. 如果两个操作数一个是变量，一个是常量，只有一种情况是不允许的，就是**常量是float，而变量是整型值**，则会提示 constant float_var truncated to integer 。这是个编译器给到的提示。为什么变量是float，常量是整型值就可以，换下就不行呢？问题就在于编译器在编译期间是确定的知道常量是float无法转成int的(因为精度会丢失的)，所以它阻止你这样做。而float变量，编译器不知道它不能表示为整数。所以让你通过。在glang的规范里有相关规定：<br>
    The values of typed constants must always be accurately representable as values of the constant type.(类型常量的值必须始终能够准确地表示为常量类型的值) <br>
    When converting a floating-point number to an integer, the fraction is discarded (truncation towards zero). (将浮点数转换为整数时，小数部分将被丢弃（向零截断）). <br>
除此之外都是可以直接计算的。

{{< highlight golang >}}
package main

import (
    "fmt"
)

func main() {
    fmt.Printf("除法--两个操作数都是整型\n")
    fmt.Printf("%T,%[1]v\n",5/4)
    fmt.Printf("%T,%[1]v\n",5/-4)
    fmt.Printf("%T,%[1]v\n",-5/4)
    fmt.Printf("%T,%[1]v\n",-5/-4)
    fmt.Printf("除法--有操作数是浮点型\n")
    fmt.Printf("%T,%[1]v\n",5/1.4)
    fmt.Printf("%T,%[1]v\n",5.1/4)
    fmt.Printf("%T,%[1]v\n",5.1/1.4)
    fmt.Printf("取模--两个操作数都是整型\n")
    fmt.Printf("%T,%[1]v\n",5%4)
    fmt.Printf("%T,%[1]v\n",5%-4)
    fmt.Printf("%T,%[1]v\n",-5%4)
    fmt.Printf("%T,%[1]v\n",-5%-4)
    fmt.Printf("取模--有操作数是浮点型\n")
    //fmt.Printf("%T,%[1]v",5%1.4)  //illegal constant expression: floating-point % operation
    //fmt.Printf("%T,%[1]v",5.1%4)  //illegal constant expression: floating-point % operation
    //fmt.Printf("%T,%[1]v",5.1%1.4)  //illegal constant expression: floating-point % operation
    fmt.Printf("--------------变量测试-------------\n")
    a,b,c,d := 5,5.1,4,1.4
    a1,b1,c1,d1 := -5,-5.1,-4,-1.4
    fmt.Printf("除法--两个操作数都是整型变量\n")
    fmt.Printf("%T,%[1]v\n",a/c)
    fmt.Printf("%T,%[1]v\n",a/c1)
    fmt.Printf("%T,%[1]v\n",a1/c)
    fmt.Printf("%T,%[1]v\n",a1/c1)
    fmt.Printf("除法--两个操作数都是变量(int and float)\n")
    //fmt.Printf("%T,%[1]v\n",a/d)  //invalid operation: a / d (mismatched types int and float64)
    //fmt.Printf("%T,%[1]v\n",b/c)  //invalid operation: b / c (mismatched types float64 and int)
    fmt.Printf("%T,%[1]v\n",b/d)
    
    fmt.Printf("取模--两个操作数都是整型变量\n")
    fmt.Printf("%T,%[1]v\n",a%c)
    fmt.Printf("%T,%[1]v\n",a%c1)
    fmt.Printf("%T,%[1]v\n",a1%c)
    fmt.Printf("%T,%[1]v\n",a1%c1)
    fmt.Printf("除法--两个操作数都是变量(int and float)\n")
    //fmt.Printf("%T,%[1]v",a%d)  //invalid operation: a % d (mismatched types int and float64)
    //fmt.Printf("%T,%[1]v",b%c)  //invalid operation: b % c (mismatched types float64 and int)
    //fmt.Printf("%T,%[1]v",b%d)  //invalid operation: b % d (operator % not defined on float64)
    fmt.Printf("除法--一个操作数都是变量(int or float)，另一个是常量(int or float)\n")
    //fmt.Printf("%T,%[1]v\n",a/1.4) //constant 1.4 truncated to integer
    fmt.Printf("%T,%[1]v\n",a/4)
    fmt.Printf("%T,%[1]v\n",5/d)
    fmt.Printf("%T,%[1]v\n",5/c)
    fmt.Printf("%T,%[1]v\n",b/1.4)
    fmt.Printf("%T,%[1]v\n",b/4)
    fmt.Printf("%T,%[1]v\n",5.1/d)
    //fmt.Printf("%T,%[1]v\n",5.1/c) //constant 5.1 truncated to integer
    
    //fmt.Printf("%T,%[1]v\n",a/(-1.4)) //constant 1.4 truncated to integer
    fmt.Printf("%T,%[1]v\n",a1/4)
    fmt.Printf("%T,%[1]v\n",5/d1)
    fmt.Printf("%T,%[1]v\n",-5/c)
    fmt.Printf("%T,%[1]v\n",b/(-1.4))
    fmt.Printf("%T,%[1]v\n",b1/4)
    fmt.Printf("%T,%[1]v\n",(-5.1)/d1)
    //fmt.Printf("%T,%[1]v\n",5.1/c) //constant 5.1 truncated to integer
}
{{< /highlight >}}

输出结果:
{{< highlight txt >}}
除法--两个操作数都是整型
int,1
int,-1
int,-1
int,1
除法--有操作数是浮点型
float64,3.5714285714285716
float64,1.275
float64,3.642857142857143
取模--两个操作数都是整型
int,1
int,1
int,-1
int,-1
取模--有操作数是浮点型
--------------变量测试-------------
除法--两个操作数都是整型变量
int,1
int,-1
int,-1
int,1
除法--两个操作数都是变量(int and float)
float64,3.642857142857143
取模--两个操作数都是整型变量
int,1
int,1
int,-1
int,-1
除法--两个操作数都是变量(int and float)
除法--一个操作数都是变量(int or float)，另一个是常量(int or float)
int,1
float64,3.5714285714285716
int,1
float64,3.642857142857143
float64,1.275
float64,3.642857142857143
int,-1
float64,-3.5714285714285716
int,-1
float64,-3.642857142857143
float64,-1.275
float64,3.642857142857143
{{< /highlight >}}
