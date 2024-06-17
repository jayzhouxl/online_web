+++
author = "jayzhou"
title = "Golang基础--一篇文章记住slice的注意点"
date = "2024-06-17"
description = "golang基础介绍第六篇,介绍golang的切片的注意点"
featured = true
tags = [
    "golang",
    "golang基础",
    "slice"
]
categories = [
    "golang",
    "slice",
]
series = ["golang","golang基础"]
aliases = ["slice"]
thumbnail = "images/golang_slice.png"
+++

本文主要深入探讨了golang的切片基础内容。阅读本篇文章后，你将对切片有更深刻的理解，并能记住在使用切片过程中需注意的要点。
<!--more-->
golang的切片用法基本和数组一致，可以根据下标存取数据，也可以通过遍历获取里面的所有的数据。但是它也有自己特有的用法和性质。本文的目的是通过更深层次理解切片，从而用好切片。
## 重要特性
1. 切片就是一块可复用的内存区域 + 一个三元组(pointer,len,cap)(切片的本质)。
2. 切片中能取出多少数，取决于切片的长度，能放多少数，取决于切片的容量。
3. 切片如果容量足够，则不会新建一个底层数组，仍会用原来的底层数组，这时候就会影响到共用底层数组的其他切片。
## 切片的内部表示
切片是基于数组实现的，它的理念跟动态数组差不多。但是切片并不是动态数组，更不是一个数组指针。它比数组指针更好用，比如在传参的时候无需传递指针或者数组的拷贝便可以读取数组以及修改数组。先看下底层结构：
{{< highlight golang >}}
type slice struct {
  array unsafe.Pointer
  len     int
  cap     int
}
{{< /highlight >}}
切片的结构体由3部分构成，len 代表当前切片的长度，cap 是当前切片的容量，array 是指向一个数组的指针。这整个结构用一句话说明，**就是指针指向一块连续的空间，len和cap表示这个这个切片用了这块空间的多少以及可以用多少**。
![切片三元组](/images/slice1.png)
是不是很简单，甚至我们可以自己建造一个这样的三元组
{{< highlight golang >}}
var s []int
sliceHeader := (*reflect.SliceHeader)((unsafe.Pointer(&s)))
sliceHeader.Cap = length
sliceHeader.Len = length
sliceHeader.Data = uintptr(ptr)
{{< /highlight >}}
这里要注意的就是，pointer所指向的空间，**是多个切片可以共用的**。切片所有的读取与写入其实都是围绕这个结构体表示的三元组来进行。

## 切片的声明和初始化
切片的声明样式和普通变量的声明一样，可以在[这里](https://www.zxlworld.com/post/golang/)看到golang的不同声明形式，但是切片的初始化方式有好几种。这里会逐一介绍这几种初始化方式，读者可以结合图片来感受len和cap的变化。
### 1.字面量初始化
{{< highlight golang >}}
s := []int{1,2,3}
var s = []int{1,2,3}
{{< /highlight >}}
和创建数组非常像，只不过不用指定[]中的值，这时候切片的**长度和容量是相等的**，并且会根据我们指定的字面量推导出来。拿例子来说，这两个切片的容量和长度都是3。
![字面量初始化](/images/slice2.png)
当然我们也可以像数组一样，只初始化某个索引的值：
{{< highlight golang >}}
slice:=[]int{1:1}
{{< /highlight >}}
这是指定了第2个元素为1，其他元素都是默认值0。这时候，**切片的长度和容量即为所指定索引的长度**。
![索引值初始化](/images/slice3.png)
### 2.make初始化
slice的make初始化也有两种形式:
{{< highlight golang >}}
s := make([]int,3)
s := make([]int,3,6)
{{< /highlight >}}
![make初始化](/images/slice4.png)
这两种形式都会去创建底层数组，唯一的区别就是是不是显式的指定容量。将make形式进行抽象:
{{< highlight golang >}}
make([]type, len, cap)
{{< /highlight >}}
如果不显式指定cap，则cap和len一致，这里的cap也就是底层数组的大小。这里的容量(cap)一定要大于等于长度(len)。在创建切片时，如果不指定字面值的话，默认值就是数组的元素的零值。
### 3.在其他数组/切片的基础上声明和初始化切片
这种初始化方式，笔者认为是最能说明切片本质的一种形式，如果能理解这种初始化形式，可以算彻底了解了切片。
{{< highlight golang >}}
s1 := []int{1,2,3,4,5,6}
s2 := s1[1:2:3]
{{< /highlight >}}
现在我们结合上面的字面量以及make方式想一想，s2的长度是多少，容量是多少，array指向哪里？结合上面的字面量以及make形式，我们大体能猜出1代表array指向，2与长度相关，3代表了容量。我们还是抽象下:
{{< highlight golang >}}
s := slice[i:j:k]
{{< /highlight >}}
这里的新切片的array指向slice[i]处，但是长度并不是j，而是j-i(所以新数组不包含j处数据)。容量也不是k，而是k-i。<br>
![本质初始化5](/images/slice5.png)
这种初始化形式用一句话来说，从原切片的i处开始，到j处结束(不包含j)，最大容量到k处(不包含k)。从这里我们可以看出，新切片来源于原切片，所以新切片的起始处，长度和容量都不能超过原切片的大小。这里最重要的一点就是新切片的数据都来源于原切片/数组，更确切的来说来源于原有的一块连续空间。可以在这个原有切片上生成多个切片
{{< highlight golang >}}
s1 := []int{1,2,3,4,5,6}
s2 := s1[1:2:3]
s3 := s1[1:3:5]
s4 := s1[2:3:5]
{{< /highlight >}}
![slice_6](/images/slice6.png)
由于不同切片用的都是同一个底层数据，所以它们的在某些位置上的数据很可能都是一样的，而唯一能区别不同切片的，就是这个三元组(pointer,len,cap)。从这个例子可以深刻的感受到文章开头说的重要特性的第一点，切片就是对底层数组复用以及能唯一识别切片的三元组。<br>
从这里也可以看出切片是灵活的，它有三个元素可以去表示它。同我们上面使用的初始化一样，我们也可以不指定容量，而是这么初始化:
{{< highlight golang >}}
s := array[i:j]
{{< /highlight >}}
这时候**切片的容量与原切片的容量一致**。这里也可以省去i，这时候默认就从数组的索引0开始，也可以省去j，默认就是原数组的长度。也可以两个都省略，那就是原数组的全部数据，容量和长度也和原数组保持一致。<br>
复用带来最大的好处就是避免了内存的重复分配，我们可以在for循环外面声明一个切片，然后重复利用这块空间，每次用完都将其长度置0，这样就需不要每次都去申请空间。还有一种用法，就是多个切片数据是一样，只有开头和结尾不一致，这样就可以在相同数据上，分别按照自己的述求决定起始地址以及长度。
当然，切片这种用法有时也会带来问题。由于不同切片共用空间，其中一个切片改了数据，就很可能会影响到另一个切片的内容。这个在下一节append里面会细讲。
### 4.nil切片和空切片
{{< highlight golang >}}
s := []int{}  //这是空切片
var s []int{}  //这是nil切片
{{< /highlight >}}
这两个切片的三元组里，len和cap都是0，唯一有区别的就是array这个指针。nil切片的指针为nil，是个空指针，而空切片的指针指向一个固定的内存地址，这是一个空数组地址，所有的空数组都指向这里，但这个地址并没有分配任何空间。
![空切片提醒](/images/slice7.png)
它们在功能上是等效的，对我们使用来说，如无特殊需求，全部都用空切片，这也是golang官方建议的用法，有些IDE，如果你用了空切片的声明，它会建议你换成nil切片声明[方式](https://www.jetbrains.com/help/go/code-inspections-in-go.html#Security)。
![slice_8](/images/slice8.png)
golang这里也对这做了说明[golang的说明](https://go.dev/wiki/CodeReviewComments#declaring-empty-slices)
![slice_9](/images/slice9.png)
意思就是除了一些特殊序列化的时候这两种会有差异以外，其余都建议使用nil切片声明方式。
### 5.底层实现
这里主要看下make的底层代码，如果读者不感兴趣，可以跳过。
{{< highlight golang >}}
func makeslice(et *_type, len, cap int) slice {
  // 根据切片的数据类型，获取切片的容量
  maxElements := maxSliceCap(et.size)
  // 比较切片的长度，长度值域应该在[0,maxElements]之间
  if len < 0 || uintptr(len) > maxElements {
    panic(errorString("makeslice: len out of range"))
  }
  // 比较切片的容量，容量值域应该在[len,maxElements]之间
  if cap < len || uintptr(cap) > maxElements {
    panic(errorString("makeslice: cap out of range"))
  }
  // 根据切片的容量申请内存
  p := mallocgc(et.size*uintptr(cap), et, true)
  // 返回申请好内存的切片的首地址
  return slice{p, len, cap}
}
{{< /highlight >}}
{{< highlight golang >}}
func makeslice64(et *_type, len64, cap64 int64) slice {
  len := int(len64)
  if int64(len) != len64 {
  	panic(errorString("makeslice: len out of range"))
  }
  
  cap := int(cap64)
  if int64(cap) != cap64 {
  	panic(errorString("makeslice: cap out of range"))
  }
  
  return makeslice(et, len, cap)
}
{{< /highlight >}}
这两个函数基本一样，int64的版本只是多了个转换。
## append 添加数据
append函数可以为一个切片追加一个元素，至于如何增加、返回的是原切片还是一个新切片、长度和容量如何改变这些细节，append函数都会帮我们自动处理。
{{< highlight golang >}}
slice = append(slice,10)
{{< /highlight >}}
append会返回值，我们需要接受该返回值，不能直接append(slice,10)，因为很可能在append的时候发生扩容，这时append会创建一个新的切片，然后在新切片的基础上进行添加，所以老切片并不会增加数据。当然如果你不接受append的返回值，编译的时候也会报错("evaluated but not used")。<br>
append的一个例子:
{{< highlight golang >}}
slice := []int{1, 2, 3, 4, 5}
newSlice := slice[1:3]
	
newSlice=append(newSlice,10)
fmt.Println(newSlice)
fmt.Println(slice)
{{< /highlight >}}
在上面介绍切片复用的时候，说过复用可能会带来原切片数据被更改的问题，上面这个例子可以很好的说明这个问题。现在我们来看下newslice的输出:
{{< highlight txt >}}
[2 3 10]
[1 2 3 10 5]
{{< /highlight >}}
可以发现原切片的数据被更改了，结合切片的三元组，newslice的三元组如下
![切片三元组](/images/slice10.png)
当我们往里添加一个数据的时候，这个时候由于切片的容量还够，所以不会触发扩容，仍在原切片的基础上进行添加
![切片append](/images/slice11.png)
这个时候就会更改掉原数组的内容。具体来说，当我们试图在newSlice后追加一个元素10时，由于newSlice有剩余容量，它直接在底层数组的下一个可用位置插入了这个值。由于newSlice和slice共享这个底层数组，所以slice的第4个元素（即底层数组的相应位置）也被修改为了10。<br>
这里要注意的是，我们所说的“newSlice新追加的第3个元素”实际上是相对于newSlice的索引而言，但在底层数组中，它实际上是第4个位置。因此，这次追加操作实际上是将底层数组的第4个元素修改为了10，并将newSlice的长度调整为3。<br>
然而，当append函数发现切片的底层数组没有足够的容量来容纳新的元素时，它会启动扩容策略。这时会创建一个新的底层数组，然后将原数组的值复制到新数组中，并追加新值。在这种情况下，由于新数组与原数组是分开的，所以对新数组的修改不会影响原数组或与之相关的其他切片。<br>
因此，为了避免因共用底层数组而引发的更改原切片的数据，我们在创建新切片时，最好确保新切片的长度和容量相等。这样，在追加操作时，append函数会创建一个新的底层数组，与原数组分离，从而避免了对原数组或相关切片的意外修改。<br>
## 切片的扩容策略
这一节主要探究golang切片底层扩容策略，不感兴趣的读者亦可跳过。
{{< highlight golang >}}
func growslice(et *_type, old slice, cap int) slice {
  if raceenabled {
    callerpc := getcallerpc(unsafe.Pointer(&et))
    racereadrangepc(old.array, uintptr(old.len*int(et.size)), callerpc, funcPC(growslice))
  }
  if msanenabled {
    msanread(old.array, uintptr(old.len*int(et.size)))
  }
  
  if et.size == 0 {
    // 如果扩容的容量比原来的容量还要小，这代表要缩容了，那么可以直接报panic了。
    if cap < old.cap {
      panic(errorString("growslice: cap out of range"))
    }
    // 如果当前切片的大小为0，就新生成一个新的容量的切片返回。
    return slice{unsafe.Pointer(&zerobase), old.len, cap}
  }

  // 扩容策略
  newcap := old.cap
  doublecap := newcap + newcap
  if cap > doublecap {
    newcap = cap
  } else {
    if old.len < 1024 {
      newcap = doublecap
    } else {
      // Check 0 < newcap to detect overflow
      // and prevent an infinite loop.
      for 0 < newcap && newcap < cap {
      	newcap += newcap / 4
      }
      // Set newcap to the requested cap when
      // the newcap calculation overflowed.
      if newcap <= 0 {
      	newcap = cap
      }
    }
  }
  
  // 计算新的切片的容量以及长度。
  var lenmem, newlenmem, capmem uintptr
  const ptrSize = unsafe.Sizeof((*byte)(nil))
  switch et.size {
  case 1:
    lenmem = uintptr(old.len)
    newlenmem = uintptr(cap)
    capmem = roundupsize(uintptr(newcap))
    newcap = int(capmem)
  case ptrSize:
    lenmem = uintptr(old.len) * ptrSize
    newlenmem = uintptr(cap) * ptrSize
    capmem = roundupsize(uintptr(newcap) * ptrSize)
    newcap = int(capmem / ptrSize)
  default:
    lenmem = uintptr(old.len) * et.size
    newlenmem = uintptr(cap) * et.size
    capmem = roundupsize(uintptr(newcap) * et.size)
    newcap = int(capmem / et.size)
  }
  
  // 非法值判断，容量确保是在增加，并且容量不超过最大容量
  if cap < old.cap || uintptr(newcap) > maxSliceCap(et.size) {
    panic(errorString("growslice: cap out of range"))
  }
  
  var p unsafe.Pointer
  if et.kind&kindNoPointers != 0 {
    // 在老的切片后面继续扩充容量
    p = mallocgc(capmem, nil, false)
    // 将 lenmem 这个多个 bytes 从 old.array地址 拷贝到 p 的地址处
    memmove(p, old.array, lenmem)
    // 先将 P 地址加上新的容量得到新切片容量的地址，然后将新切片容量地址后面的 capmem-newlenmem 个 bytes 这块内存初始化。为之后继续 append() 操作腾出空间。
    memclrNoHeapPointers(add(p, newlenmem), capmem-newlenmem)
  } else {
    // 重新申请新的数组给新切片
    // 重新申请 capmen 这个大的内存地址，并且初始化为0值
    p = mallocgc(capmem, et, true)
    if !writeBarrier.enabled {
      // 如果还不能打开写锁，那么只能把 lenmem 大小的 bytes 字节从 old.array 拷贝到 p 的地址处
      memmove(p, old.array, lenmem)
    } else {
      // 循环拷贝老的切片的值
      for i := uintptr(0); i < lenmem; i += et.size {
    	typedmemmove(et, add(p, i), add(old.array, i))
      }
    }
  }
  // 返回最终新切片，容量更新为最新扩容之后的容量
  return slice{p, old.len, newcap}
  }
{{< /highlight >}}
翻译过来就是:<br>
1. 判断新容量是否大于旧容量的两倍：<br>
如果新申请的容量（cap）大于旧切片容量（old.cap）的两倍，则最终容量（newcap）就是新申请的容量（cap）。否则判断原切片的长度。<br>
2.1. 原切片长度小于1024：<br>
如果旧切片的长度（old.len）小于1024，并且新申请的容量不大于旧容量的两倍，则最终容量（newcap）就是旧容量（old.cap）的两倍，即 newcap = old.cap * 2。<br>
2.2. 原切片长度大于等于1024：<br>
如果旧切片的长度（old.len）大于等于1024，则最终容量（newcap）从旧容量（old.cap）开始，每次增加旧容量的四分之一，即 newcap += newcap / 4，直到最终容量（newcap）大于等于新申请的容量（cap），即 newcap >= cap。<br>
3. 容量溢出检查：<br>
在计算最终容量（newcap）时，Go 运行时还会检查是否会发生整数溢出。如果计算出的 newcap 超过了 int 类型的最大值，则最终容量（newcap）将等于新申请的容量（cap)。<br>

这种扩容策略是为了在性能和空间利用之间找到一个平衡点。当切片较小时，通常将容量翻倍，以避免频繁的内存分配和复制操作。但当切片变得非常大时，翻倍扩容可能会导致浪费过多的内存空间，因此采用了逐渐增加的策略。
## 切片的拷贝(copy)
切片copy的时候只会拷贝目标长度的数据，超过目标长度的不拷贝，注意这里是长度而不是容量
{{< highlight golang >}}
func main() {
    slice := []int{1, 2, 3, 4, 5}
    slice2 := make([]int,0,100)
    copy(slice2,slice)
    fmt.Printf("长度为0,数据:%+v,len:%d,cap:%d\n",slice2,len(slice2),cap(slice2))
    slice3 := make([]int,3)
    copy(slice3,slice)
    fmt.Printf("长度不为0,数据:%+v,len:%d,cap:%d\n",slice3,len(slice3),cap(slice3))
}
{{< /highlight >}}
{{< highlight txt >}}
长度为0,数据:[],len:0,cap:100
长度不为0,数据:[1 2 3],len:3,cap:3
{{< /highlight >}}
一句话来说，**从源切片的起始处拷贝目标切片长度的数据**。<br>
再看下copy的源代码，不感兴趣，亦可跳过。
{{< highlight golang >}}
func slicecopy(to, fm slice, width uintptr) int {
  // 如果源切片或者目标切片有一个长度为0，则直接return
  if fm.len == 0 || to.len == 0 {
    return 0
  }
  // 记录源切片或者目标切片的长度较小值
  n := fm.len
  if to.len < n {
    n = to.len
  }
  // 如果入参为0，也不需要拷贝，直接返回较短的切片长度
  if width == 0 {
    return n
  }
  // 竞争检测判断
  if raceenabled {
    callerpc := getcallerpc(unsafe.Pointer(&to))
    pc := funcPC(slicecopy)
    racewriterangepc(to.array, uintptr(n*int(width)), callerpc, pc)
    racereadrangepc(fm.array, uintptr(n*int(width)), callerpc, pc)
  }
  if msanenabled {
    msanwrite(to.array, uintptr(n*int(width)))
    msanread(fm.array, uintptr(n*int(width)))
  }
  
  size := uintptr(n) * width
  if size == 1 { 
    d// TODO: is this still worth it with new memmove impl?
    // 如果只有一个元素，那么直接转换指针即可
    *(*byte)(to.array) = *(*byte)(fm.array) // known to be a byte pointer
  } else {
    // 如果不止一个元素，那么就把 size 个数据 从 fm.array 地址开始，拷贝到 to.array 地址之后
    memmove(to.array, fm.array, size)
  }
  return n
}
{{< /highlight >}}
## 切片的遍历
切片是一个集合，我们可以使用 for range 循环来迭代它，打印其中的每个元素以及对应的索引。
{{< highlight golang >}}
  slice := []int{1, 2, 3, 4, 5}
  for i,v:=range slice{
    fmt.Printf("索引:%d,值:%d\n",i,v)
  }
{{< /highlight >}}
如果我们不想要索引，可以使用_来忽略它，这是Go语言的用法，很多不需要的函数等返回值，都可以忽略。
{{< highlight golang >}}
slice := []int{1, 2, 3, 4, 5}
for _,v:=range slice{
  fmt.Printf("值:%d\n",v)
}
{{< /highlight >}}
这里需要说明的是golang这里的遍历，从始至终都只有一个变量，整个遍历的过程就是不断的往这个变量中复制数据。
{{< highlight golang >}}
func main() {
    s := []int{1,2,3}
    var wg sync.WaitGroup
    for _,v := range s {
        wg.Add(1)
        go func() {
            fmt.Printf("%d\n",v)
            wg.Done()
        }()
    }
    wg.Wait()
}
{{< /highlight >}}
这里会全都输出3，每个golang程序员应该都遇到过这个困惑。主要就是golang的for 中循环变量的定义是 per loop 而非 per iteration，也就是整个 for 循环期间，变量 i 只会有一个。上面这个例子输出都是3还有一个原因是因为协程启动的速度太快，从而导致都是拿的for循环最后的那个v。再举个更明显的例子:
{{< highlight golang >}}
func main() {
    s := []int{1,2,3}
    s2 := make([]*int,0)
    for _,v := range s {
       s2 = append(s2,&v)
    }
    for _,v := range s2 {
        fmt.Printf("%d\n",*v)
    }
}
{{< /highlight >}}
这里s2拿的其实都是同一个地址，而这个地址在第一个for循环最后的时候被塞入了3，所以第二个for循环都输出3。<br>
这个问题其实在 C++ 中也同样[存在](https://wandbox.org/permlink/Se5WaeDb6quA8FCC)。但真的太容易搞错了，几乎每个 Go 程序员都踩过一遍，而且也非常容易忘记。即使这次记住了，下次很容易又会踩一遍。
甚至知名证书颁发机构 Let’s Encrypt 就踩过一样的坑 [bug#1619047](https://bugzilla.mozilla.org/show_bug.cgi?id=1619047)。<br>
不过golang1.22版本将要修复这个问题了。
## 总结
1. 切片就是个三元组，多个切片可以共用底层数组。
2. append数据时，如果原切片容量足够，则不会新增切片，这个时候就可能会更改原有切片数据。
3. for循环遍历切片的问题需要注意只有一个变量存在。
