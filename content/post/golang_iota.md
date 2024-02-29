+++
author = "jayzhou"
title = "Golang基础--只此一篇,立马记住iota用法"
date = "2024-02-29"
description = "golang基础介绍第四篇,立马记住iota和const用法"
featured = true
tags = [
    "golang",
    "golang基础",
    "iota",
    "const"
]
categories = [
    "golang",
    "iota",
    "const",
]
series = ["golang","golang基础"]
aliases = ["iota","const"]
thumbnail = "images/golang_iota.png"
+++

本文主要是对Golang中iota和const的用法,做了总结,目的就是让你通过这一篇文章就能立马知道iota的用法.
<!--more-->
在寻找一篇简洁明了的文章，来帮助你理解iota的作用？如果是这样，那么你来对地方了。在这篇文章中，我将尽可能简洁地解释iota和const特性，并通过实例来展示它们的应用。<br>
## 太长不看版
iota和const的主要特性可以归纳为以下两点：<br>
1. 在const块中，const会隐式重复前一个非空表达式，但要注意要与前一行常量(=号前面的标识符)数量一致。
2. iota代表的是所在const行的行号。(这个特性,网上好多复制粘贴的文章的文章都说错了,不是说iota遇到const就变为0).

## 深入探索
让我们通过一个例子来理解这些特性。考虑以下的Golang代码：

{{< highlight golang >}}
package main

import (
    "fmt"
)

const (
    i=1<<iota
    j=3<<iota
    k
    l
)

func main() {
   fmt.Println("i=",i)
   fmt.Println("j=",j)
   fmt.Println("k=",k)
   fmt.Println("l=",l)
}
{{< /highlight >}}

输出结果:
{{< highlight txt >}}
i= 1
j= 6
k= 12
l= 24
{{< /highlight >}}
其实上面的表达式变成了如下形式：
{{< highlight txt >}}
i=1<<0
j=3<<1
k=3<<2
l=3<<3
{{< /highlight >}}
为什么会这样呢？这其实牵扯到两方面，一个是const的特性，另一个是auto的特性，接下来详细说说。
## const特性：隐式重复前一个非空表达式
Go的const语法有一个隐藏的机制，那就是隐式的重复前一个非空表达式, 这意味着，如果我们在const声明语句块中省略了某个常量的值，那么该常量将使用前一个非空表达式的值。
{{< highlight golang >}}
const (
    Apple, Banana =  1,  2
    Cherimoya, Durian
    Elderberry,Bole
)
{{< /highlight >}}
Cherimoya, Durian和Elderberry,Bole 输出都是1，2，其实就是变成了下面这几个形式:
{{< highlight golang >}}
const (
    Apple, Banana =  1,  2
    Cherimoya, Durian = 1, 2
    Elderberry,Bole = 1,  2
)
{{< /highlight >}}
这是golang语法提供的一种语法糖。但是这里要注意，表达式 "=" 前后缺省的常量要一致，也就是说下面这些写法，是不被允许的：
{{< highlight golang >}}
const (
    Apple, Banana =  1,  2
    Cherimoya, Durian=3
    Elderberry,Bole
)
const (
    Apple, Banana =  1,  2
    Cherimoya, Durian
    Elderberry
)
const (
    Apple, Banana =  1,  2
    Cherimoya
    Elderberry,Bole
)
{{< /highlight >}}
但是下面这些是可以的
{{< highlight golang >}}
const (
    Apple, Banana =  1,  2
    Cherimoya, Durian=3, 4
    Elderberry,Bole
)
const (
    Apple, Banana =  1,  2
    Cherimoya, Durian
    Elderberry=3
)
const (
    Apple, Banana =  1,  2
    Cherimoya=3
    Elderberry,Bole = 3,4
)
{{< /highlight >}}
总结来说就是下一行的常量数量要与上一行一致，常量缺省的数量也要一致，要么都没有，不能一个有一个没有。<br>
到这里，我们看上面的例子，其实发现，const这个特性好像没什么意义，只是简单的重复。但如果结合下面要讲的iota的特性，就变得很有用了。
## iota特性:iota表示的是这个字段在const中行号
首先我们要知道iota只能在const中使用。iota表示的是他所在的行数,这个与网上的文章说的是不一样的.
{{< highlight golang >}}
const (
  b = iota     //b=0
  c            //c=1
)
{{< /highlight >}}
这个大家都知道b等于0，那如果将iota放到下一行呢?
{{< highlight golang >}}
const (
  b = 0     //b=0
  c = iota           //c=1
)
{{< /highlight >}}
c 等于1，再往下移一行：
{{< highlight golang >}}
const (
  b = 2     //b=0
  c = 5           //c=1
  d =iota
)
{{< /highlight >}}
d 输出3。注意这里我们将b，c的输出换了，不再是前面的跟行数重合的数字，避免混淆。最终iota所赋值的仍然输出了它所在的行数。<br>
这个跟网上说的iota遇到const就为0，是不一样的。**iota代表了这个const常量值所在的位置(行数)**。这个要记住. <br>
上面的例子中，我们看到每行只有一个常量，下面我们看看每行有多个常量的情况。 <br>
{{< highlight golang >}}
const (
  b,b1 = iota,iota+1     //b=0
  c,c1            //c=1
)
{{< /highlight >}}
c1为3，b1为2. <br>
刚刚说了iota代表了它所在行的序号，所以即使你在同一行上写了两个iota，它所代表的都是当前行的序号。<br>
现在将const和iota的这两个特性结合起来，再来看个const的例子：
{{< highlight golang >}}
const (
    Apple, Banana =  iota+1,  iota+2
    Cherimoya, Durian
    Elderberry,Bole
)
{{< /highlight >}}
由于const特性，所以Cherimoya和Durian就变成了Cherimoya, Durian = iota+1,iota+2.再结合iota的特性，其实就是Cherimoya, Durian = 1+1,1+2。这样将const和iota两个特性结合起来，就可以省去我们重复书写的工作量，方便我们定义const常量。
## 总结：
1. const会隐式重复前一个非空表达式，但要注意要与前一行常量数量(=号前面)一致,同一行常量的缺省设置也要一致,不能有的有值,有的没有值。
2. iota代表的是所在const行的行号。这个特性跟网上所说的文章都不一样,不是说iota遇到const就变为0,大家一定要注意区别.
只要记住这两个特性，就完全可以理解iota的作用了。不用再去看网上的那些繁复而且是错误的文章。<br>

