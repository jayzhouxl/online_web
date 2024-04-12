+++
author = "jayzhou"
title = "Golang基础--一篇文章记住switch的所有用法"
date = "2024-04-11"
description = "golang基础的第五篇文章--switch"
featured = true
tags = [
    "golang",
    "golang基础",
    "总结",
    "switch"
]
categories = [
    "golang",
    "switch",
]
series = ["golang","golang基础"]
aliases = ["switch"]
thumbnail = "images/golang_switch.png"
+++

本篇文章对golang中switch的用法做了总结,可以通过该篇文章快速记住switch的全部用法。
<!--more-->

## 速记
1. switch只有一个作用，那就是匹配true。
2. 匹配上某个case后直接退出匹配，除非使用fallthrough。ackage main
## 详细分析
### 1.switch就是用来匹配true
这一点的使用体现在三个方面，但是核心思想就是用来匹配true的，只要记住这点就都不难，下面详细说下<br>
#### a.switch 后面跟表达式，
{{< highlight golang >}}
switch expr {
  case value:
}
{{< /highlight >}}
这里主要就是将表达式后case后面的value进行一一比对，如果相等，就命中该case。可以<br>
 expr == value
这么记忆。也就是判断上述表达式是否为true
##### 两点注意：
1. expr 和 value 不局限于数字，字符串也行，只要保证类型一致。
{{< highlight golang >}}
switch str {
  case "abc":
    fmt.Println("Hello world!")
}
{{< /highlight >}}

2. 后面的表达式是产生值的，不可以是赋值语句，也就是要能组成 expr == value 这个判断条件。
### 2.switch 后面不跟表达式，那这个时候就需要case后面自己能进行是否为true的判断。
{{< highlight golang >}}
switch {
  case expr == value:
}
{{< /highlight >}}
这里的case后面的其实就是 true或者 false。
{{< highlight golang >}}
switch {
  case true:
}
{{< /highlight >}}
### switch 用来进行类型的判断
这个也是swtich用的比较多的地方。这里的用法稍稍有点不一样，但是核心还是判断是否为true
{{< highlight golang >}}
func do(i interface{}) {
  switch v := i.(type) {
    case int:
      fmt.Printf("Twice %v is %v\n", v, v*2)
    case string:
      fmt.Printf("%q is %v bytes long\n", v, len(v))
    default:
      fmt.Printf("I don't know about type %T!\n", v)
  }
}
{{< /highlight >}}
这里就是将i的类型与case后面的类型判断是否为true，可以这么记忆<br>
i.(type) == int ，同时还能拿到对应的值。
### switch只有加上fallthrough 才会继续跳到下一个case。
这个是跟C++不一样的地方，我觉得也是符合常识的，因为我命中了case，只想走这个case，其他case一般都不用走，golang这种用法，就不需要每个case再写一个break了。如果想继续走接下来的case(这种情况算少数)，则加个fallthrough就行了。
{{< highlight golang >}}
str := "1"
switch str {
  case "1":
    fmt.Println("1")
  case "2":
    fmt.Println("2")
}
{{< /highlight >}}
输出结果:
{{< highlight txt >}}
1
{{< /highlight >}}
{{< highlight golang >}}
str := "1"
switch str {
  case "1":
    fmt.Println("1")
    fallthrough  // 注意: 放在语句的下一行
  case "2":
    fmt.Println("2")
}
{{< /highlight >}}
输出结果:
{{< highlight txt >}}
1
2
{{< /highlight >}}

**以上golang用法以及用例在golang版本 1.19.1 均测试通过**
