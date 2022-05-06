---
title: 用Go刷LeetCode和OJ
date: 2021-10-8 0:0:0
tags:
- go
abbrlink: 'go-oj'
hide: true
---
生计所迫，开始庸俗的刷题，要恰饭的嘛，要找工作的嘛。下面这些假设你熟悉c/c++等其他语言的一些基本语法，简单说一下go对应的操作和相应的库（没有STL刷题体验还是不太好的）
<!-- more -->

我一直比较喜欢的是c，后来喜欢上了go。对c++一直无感（c++ 20 花样太多了，给我一种python那样回字的八种写法的感觉）

最后发现还是C++适合刷题，go刷题不少数据结构没有STL好累， 咳咳。（Go最舒服的是拿来写并发和网络这些东西，然而好几个平台多线程的题，比如哲学家就餐问题，都不支持Go）

## 输入输出

Go的输入输出跟c差不多。

之前看到HackerNews上讨论一个打印`yes`的命令的速度，已经水过一篇go的输出速度了，这里简单说一下OJ的输入输出。主要的库有`fmt`， `bufio`， `strconv`, `io`。如果你不用`fmt`直接用内置的`println()`也不是不可以。

我们随便打开一个支持Golang的OJ平台，比如[UESTC OJ](https://acm.uestc.edu.cn/), 输入输出测试题A+B

```go
package main

import (
	"fmt"
	"io"
)

func main() {
	var a, b int
	for {
		if _, err := fmt.Scan(&a, &b); err != io.EOF {
			fmt.Println(a + b)
		} else {
			break
		}
	}
}
// 或者我们为了省事，不import "io"，直接if _, err := fmt.Scan(&a, &b); err == nil
// 或者为了省事我们写成这样，不import "io", 直接用 n == 0 判断是否结束（工程上不要这么干）
// func main() {
// 	var a, b int
// 	for {
// 		n, _ := fmt.Scan(&a, &b)
// 		if n != 0 {
// 			fmt.Println(a + b)
// 		} else {
// 			break
// 		}
// 	}
// }

//不加else看起来更清爽一点
// for {
//     n, _ := fmt.Scan(&a, &b)
//     if n == 0 {
//         break
//     }
//     fmt.Println(a + b)
// }
```

如果是刷惯了leetcode临时抱佛脚练习输入输出可以去这里 [牛客 OJ在线编程常见输入输出练习](https://ac.nowcoder.com/acm/contest/5657)  

比较常用的是输入多行，用回车分隔。这种我喜欢用`bufio`直接读一行然后处理(字符串转换可能会有一点效率问题，一般问题不大)

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strconv"
	"strings"
	"sort"
)

// A+B第四题
func AaddB() {
	buf := bufio.NewScanner(os.Stdin)
	for buf.Scan() {
		arr := strings.Split(buf.Text(), " ")
		if arr[0] == "0" {
			break
		}
		ans := 0
		for i := 1; i < len(arr); i++ {
			num, _ := strconv.Atoi(arr[i])
			ans += num
		}
		fmt.Println(ans)
	}
}

//读取字符串并排序输出
func sortString() {
	buf := bufio.NewScanner(os.Stdin)
	for buf.Scan() {
		arr := strings.Split(buf.Text(), " ")
        sort.StringSlice.Sort(arr)
        fmt.Println(strings.Join(arr, " "))
	}
}
```

### fmt

[fmt标准库](https://pkg.go.dev/fmt)的输入输出在大多数情况下的输入输出都够了。`fmt.Print()`和`fmt.Printf()`跟c里面的`print`和`printf`几乎完全一样。

`Scan`和`Print`是最简单的输入输出，空格或者回车分割输入参数。`Scan`有两个返回值，一个是Scan成功的个数，一个是error

`Scanln`和`Println`和上面两个功能类似，区别是`ln`输入回车会结束输入输出。

`Scanf`可以用来格式化输出，用法和c语言一样。

`Sprintf`有的时候直接用来拼接字符串比`strconv`好用， 比如Leetcode 401 Binary-watch，可以用`Sprintf`格式化为String然后赋值给ans

```go
ans = append(ans, fmt.Sprintf("%d:%02d", i, j))
```

`Fprintf`这些带`F`的区别是输出到`io.Writer`而`Print`是输出到标准输出`os.Stdout`，刷题的时候一般不用。

```go
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
```

### bufio

缓冲区能在有的时候加快读取速度，但是缓冲区缓存可能导致输入输出不对AC不了，请小心使用

### 标准输入输出

和c的`stdio`一样，直接调用系统的标准输入输出，这个比`fmt`里面的库要快一点

## 一些库

### container包

## 堆

堆可以用`container/heap`，提供了一个最大堆（），可以用来实现优先队列、堆排序。堆的元素可以是任意类型（自己定义一个`type`就好了）

标准库提供了Interface，需要我们自己补全`Push`和`Pop`方法。此外还有一个`sort.Interface`，需要我们自行实现`Len`, `Less`, `Swap`方法

```go
type Interface interface {
	sort.Interface
	Push(x interface{}) // add x as element Len()
	Pop() interface{}   // remove and return element Len() - 1.
}
```

例子：Leetcode 215 692 2321 502

### 内置的后缀数组

后缀数组(suffixarray, sa)就是将字符串的后缀按字典序排序，详见[后缀数组简介](https://oi-wiki.org/string/sa/)  

go的后缀数组是在标准库里面的：

```go
import "index/suffixarray"

index := suffixarray.New(s) //时间复杂度O(N*log(N))
offset := index.Lookup(s1, -1)  //查找所有s1出现的位置
```

这个包一般是用来查找的，官方只给了一个Lookup方法。用来解决一些字符串的题用到sa和rank数组，需要一些扭曲的reflect和unsafe来获得，比如1163题，suffixarray的结构是这样的：

```go
type Index struct {
	data []byte
	sa   ints // suffix array for data; sa.len() == len(data)
}

type ints struct {
	int32 []int32
	int64 []int64
}
```

所以获取sa和rank数组需要这样

```go
// 求出后缀数组sa和rank
func lastSubstring(s string) string {
	n := len(s)
    sfx := suffixarray.New([]byte(s))
	sa := *(*[]int32)(unsafe.Pointer(reflect.ValueOf(sfx).Elem().FieldByName("sa").Field(0).UnsafeAddr()))
    // 求rank，本题不需要
	// rank := make([]int, n)
	// for i := range rank {
	// 	rank[sa[i]] = i
	// }
	return s[sa[n-1]:]
}
```

### 搜索和排序

go有[sort包](https://pkg.go.dev/sort)，常见的搜索可以直接用，二分搜索也可以自定义（很方便，写二分再也不会错了）

sort包有接口可以自定义数据类型和排序规则。可以自行实现`Len`, `Less`, `Swap`方法

```go
type Interface interface{
     Len() int
     Less (i , j) bool
     Swap(i , j int)
}
```

`sort.Slice` 自定义切片排序。这个也很方便，不用再去把`Len`、`Less`、`Swap`的方法都逐个实现一遍，比如Leetcode 502 题

```go
arr := int{1,2,3,4,5}
sort.Slice(len(arr), func(i, j int) bool {
	return 
})
```

### 数学运算

go的%是取余，不是取模。

golang的数值计算库只有Float64。。。math包里面不提供自带的int等数值类型比较大小。可能是因为没有重载实现起来麻烦。当然你可以先转成Float64再转回来，不过一般的做法是自己定义一个比较大小的函数：

```go
func max(a,b int) int {
    if a > b {
        return a
    }s
    return b
}
```

数组max

```go
func max(args ...int) int {
    var ans = args[0]
    for _, v := range args {
        if v > ans {
            ans = v
        }
    }
    return ans
}
```

用还没完全支持的范型可以实现支持大部分类型的`min/max`函数

```go
type MathType interface {
    type int, int8, int16, int32, int64,
        uint, uint8, uint16, uint32, uint64, uintptr,
        float32, float64, complex64, complex128
}

func max[T MathType](a, b T) T {  
    if a > b {  
        return a 
    }
    return b
}
```

## 中间变量

回溯和迭代的时候, 作为中间变量的slice要使用Copy，直接对slice进行append是无法获取到中间结果的。

```go
copy(arr1, temp)
ans := append(ans, temp)
```

或者写成这个样子（不推荐）

```go
ans = append(ans, append([]int(nil), path...))
```

## 字符串

### string库

string这个库常用的功能几乎都有了，如果想偷懒写很多字符串的题都可以偷懒

（面试的时候应该不能偷懒，而且某跳动貌似特喜欢考字符串的题。。。）

```
# 是否包含
func Contains(s, substr string) bool
# 是否包含子串
func ContainsAny(s, chars string) bool
func Count(s, sep string) int
```

### 字符串比较

可以直接用`<`号、 `==`号、`>`号比较大小，比较结果是ASCII码的是字典顺序(跟长度没有关系，如果要比长度先用len处理一下)。

```go
s1, s2 := "abc", "abd"
fmt.Println(s1<s2)
```

`strings`库有个`*func* *Compare*(a, b string) int`和`*func* *EqualFold*(s, t string) bool`两个函数。
`strings.Compare`只是一个简单的封装而已，输出-1, 0, 1对应大于等于小于。不推荐使用，因为慢（Compare is included only for symmetry with package bytes. It is usually clearer and always faster to use the built-in string comparison operators ==, <, >, and so on.）

```go
func Compare(a, b string) int {
	if a == b {
		return 0
	}
	if a < b {
		return -1
	}
	return +1
}
```

`strings.EqualFold`函数比较相等时会忽略大小写，"=="大小写不同时返回false

### 字符串拼接

字符串可以直接用`+`号拼接，比如`str = "a" + "b" + "c"`的结果是`"abc"`。
但是这样的效率很低。因为go的string是不可变的，每次用`+`号执行拼接操作，都会新建一个字符串。于是人们为了性能干出了拼接byte然后再转成string的操作。

在1.10版本以后（注意版本问题，有的OJ平台2021年了go版本还没到1.10,很坑），有了strings.Builder等，不用再扭曲的用byte类型转来转去了。

看[builder.go](https://golang.org/src/strings/builder.go)的实现不难发现，这就是封装了`[]byte`然后加了缓冲区等东西。类型转换是直接用`unsafe.Pointer`实现的。

```go
# init
var s1 strings.Builder
# or
s2 := &strings.Builder{}

....

# finish
str := s1.String()
```

几个常用的拼接

```go
func (b *Builder) Write(p []byte) (int, error)
func (b *Builder) WriteByte(c byte) error
func (b *Builder) WriteRune(r rune) (int, error)
func (b *Builder) WriteString(s string) (int, error)
```

注意，不要拷贝非空的stirngs.Builder, 另外，这玩意不支持协程。

当然如果你完全不在乎性能的，可以使用`Sprintf`这种特别方便的格式化函数。代价是要忍受严重影响速度的io系统调用。（我一般在本地调试的时候用这个，顺手打印出来）

### 字符串类型转换

注意go特有的rune类型, 如果你用`for range`遍历字符串s，那么

int转byte要注意大小端的问题，不能直接转换。

## LeetCode本地输入

Leetcode的vscode插件很好用，但是在线判题速度很慢，也不支持调试。

偶然看到这个文章：[利用Golang反射机制(Reflect)搭建本地LeetCode调试器](https://www.aaron-xin.tech/posts/leet-go/)，用反射实现的输入输出。

## 忽略错误处理

写业务的时候错误处理一般不能忽略，但是写一些小题目每次`if err != nil {}`,行数恐怕要翻倍。一般刷题的时候我习惯用`_`忽略error

## 关于unsafe包

一般不怎么用，有的时候类型强制转换比较方便。

## 关于反射

反射会有性能问题，一般不怎么用。比较有用的是`reflect.DeepEqual`，可以用来判断struct或map是否相等（有一定的性能问题, 不如自己写的遍历快）

## 我比较喜欢的题目

450 LFU缓存

## 链接

其他小工具

https://github.com/EchoUtopia/helper/blob/master/README_CN.md

https://github.com/liyue201/gostl/blob/master/README_CN.md

一个暂时还不能用的提案：https://news.ycombinator.com/item?id=27048531 slices的新提案。

[golang-strings-builder-原理解析](https://liudanking.com/performance/golang-strings-builder-原理解析/)

https://github.com/golang/go/blob/master/src/fmt/print.go#L620

https://medium.com/@thuc/8-notes-about-strings-builder-in-golang-65260daae6e9

https://leetcode-cn.com/circle/discuss/pv7bY1/ 