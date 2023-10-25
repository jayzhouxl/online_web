+++
author = "jayzhou"
title = "Golang基础--变量声明"
date = "2023-10-08"
description = "golang基础介绍第一篇，介绍golang里面的变量声明"
featured = true
tags = [
    "golang",
    "golang基础",
    "声明",
    "定义",
    "内存分配"
]
categories = [
    "golang",
    "声明",
    "定义",
]
series = ["golang","golang基础"]
aliases = ["golang声明"]
thumbnail = "images/golang.png"
+++

本文主要是介绍Golang中变量声明的相关内容，主要涉及golang变量声明与定义的区别，声明的两种形式，以及何时该用哪种声明形式。
<!--more-->

## 声明与定义

在Golang中，声明即是定义。这个是与C++中不同的地方，在C++中声明和定义是不同的。声明是用来告诉编译器变量名称和类型，而不分配内存。定义则是给变量分配内存，可以给变量赋初值，这个还是有点容易混淆的。在Golang中，声明即是定义，当所谓的声明一个变量的时候，这个变量对应的值总是会被初始化，要么用指定的值初始化，用么用类型默认的值初始化:

{{< highlight golang >}}
package main
import "fmt"
type person struct {
	age  int
	name string
}
func main() {
	var i int
	fmt.Printf("int %p,%d\n", &i, i)
	var s string
	fmt.Printf("string %p,[%s]\n", &s, s)
	var b bool
	fmt.Printf("bool %p,%t\n", &b, b)
	var p *int
	fmt.Printf("*int %p,%p\n", &p, p)
	var m map[int]int
	fmt.Printf("map %p,%v\n", &m, m)
	var sli []int
	fmt.Printf("slice %p,%v\n", &sli, sli)
	var human person
	fmt.Printf("person %p,%+v\n", &human, human)
}
{{< /highlight >}}

输出结果:
{{< highlight txt >}}
int 0xc000090020,0
string 0xc0000721e0,[]
bool 0xc000090040,false
\*int 0xc000092020,0x0
map 0xc000092028,map[]
slice 0xc000088040,[]
person 0xc000088060,{age:0 name:}
{{< /highlight >}}
可以看出所有的变量开辟了内存空间，而且无论自定义类型还是自有类型，在定义了之后都会有默认的零值。所以在golang中声明即是定义。

## 声明的两种形式
在golang里面声明有两种形式，一种是通用的声明形式，另一种则是比较特殊的短变量声明形式。

### 通用声明形式
var name type = expression 类型和表达式可以省略一个，但是不能都省略。如果省略类型，则类型由expression决定，如果表达式省略，则初始值就是对应type的零值。
有种特殊形式的声明，与通用形式一致，但是直观上不明显:
{{< highlight golang >}}
var x struct {
  a bool
  b int16
  c []int
}
{{< /highlight >}}
这里声明了一个结构体变量，name为x，省略了表达式。

### 短变量声明
当针对有指定值的初始化时，可以看出通用形式的声明，写法还是偏复杂的，所以在golang中还有一种简短的写法：variable_name := 表达式 或 值
使用此操作符的主要目的是在函数内一个语句完成声明和将变量初始化为指定值，在这里定义的变量的类型是由表达式的类型决定，不需要人为指定。可以这种写法让变量的定义更为简洁。

### 两种声明的异同
这两种声明形式主要有两种区别：作用域以及变量的类型指定。
使用短变量声明的变量作用域只能在函数内部，也就是说这种写法只能在函数内部出现。而通用的声明形式则没有这种限制，既可以声明局部变量，也可以声明全局变量。
采用短变量申明的变量类型是由表达式或值的类型推断出来的，该变量的类型与值的类型保持一致。而通用声明的变量类型是可以自己直接指定的，可以与值的类型不一致(表达式与类型是可转换，不能将字符串转换为数字;同时类型以type为准。e.g. var i float64=100 ,i类型是float64)。

### 具体使用场景
可以看出，短变量声明的变量，肯定是设置了自定义值的，所以如果只是设置了默认零值，就不需要用短变量声明形式。任何时候，创建一个变量并并采用其默认零值作为初始值，习惯是使用关键字var这种用法。这种用法是为了更明确地表示这个个变量被设置为零值。如果变量要被初始化为某个非零值，这时候就可以配合结构字面量和短变量声明操作符来创建变量。
短变量声明只能用在函数中，var声明也可以用在声明局部变量(全局变量也可以)。初始化值类型跟变量类型不一致情况(表达式与类型是可转换，不能将字符串转换为数字;同时类型以type为准。e.g. var I float64=100 ,i类型是float64)。所以如果需要声明局部变量，或者指定变量类型，则也需要使用通用形式的声明，除此之外都可以使用短变量声明，毕竟字数少，快。

## 多个变量的声明
短变量声明以及通用形式的声明都支持同时声明多个变量:
{{< highlight golang >}}
var i,j,k int //这种声明只能声明同类型变量。
Var i,j,k = true, 2, "aa" //忽略类型允许声明多个不同类型的变量。
i, j, k := true 2, "aa" //短变量声明也支持
i,j,k = true 2, "aa" //多重赋值，这边要注意，要赋值的表达式和变量类型要一致，i要是bool，j是int，k是string
{{< /highlight >}}
