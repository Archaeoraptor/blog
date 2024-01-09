---
title: C++返回值类型后置
date: 2024-01-09 10:59:10
tags:
- type
- c
abbrlink: 'type-anastrophe'
mathjax: true
---
不喜欢倒装吗你们
<!-- more -->

类型后置的语言一般长这样，比如go，函数的返回值类型是放在后面的

```go
func mul(a, b int) int {
    return a * b
}
```

rust也是：

```rust
fn mul(a: i32, b: i32) -> i32 {
    a * b
}
```

甚至py也可以指定函数返回值类型。

```python
def mul(a: int, b: int ) -> int:
    return a*b
```

但是大家熟悉的c，函数是这样，返回值类型在最前面。

```c
int mul(int a, int b)
{
    return a * b;
}
```

反而是c语言的类型前置才像个异类。当初从c入坑go的时候，总感觉后置比较别扭，后来发现是我被c毒害的太深了：[Go's declaration syntax](https://go.dev/blog/declaration-syntax)  

这是c，返回值类型前置

```c
int (*(*fp)(int (*)(int, int), int))(int, int)
```

这是go，返回值类型甩在后面

```c
f func(func(int,int) int, int) func(int, int) int
```

写一个闭包看起来也很自然：

```go
sum := func(a, b int) int { return a+b } (3, 4)
```

写多了go可能会感觉到，go里面很少出现c语言一样一大坨*号和括号挤在一起，区分不出来这到底是个啥。函数返回值后置就比较舒服，一般写函数的时候都是返回值在后面的，lambda表达式也是。

$$
f(x, y) = x \times y
$$

对于空的返回值，扔在后面直接不写看起来也更加自然。一个void总是怪怪的。

```c
void echo() {
    printf("done");
}
```

```go
func echo() {
	println("done")
}
```

网上有相当多的文章教大家怎么识别c语言的复杂的类型定义。什么函数指着呐，什么指针函数呐。甚至有一些网站，比如[cdel](https://cdel.org)专门给你拆解这是个啥。别的语言为啥就没有呢？

后来接触了好多返回值后置的语言，我才意识到：看不c语言这一大坨东西，不是我的问题，是c的问题。。。你看下面这一串是个啥，你不说这是c我都要幻视了以为是lisp呢。

```c
// 括号乱起来了
int (*(*foo)())()
// 下面这个更有内味
(*(void(*)())0)();
```

c++11也有类型后置的写法，前面使用auto占位。比较符合我先写输入再写输出的喜欢，现在rust和go写多了经常c++一起手就在先思考一下函数返回值类型那里愣住了五秒。

```cpp
auto mul(int a, b) -> int {
    return a * b;
}
```

配合decltype可以实现类型推导。

```cpp
template<typename T1, typename T2>
decltype((*(T1 *) 0) + (*(T2 *) 0)) mul(T1 a, T2 b)
{
    return a * b;
}

// 可以用auto占位写成后置
template<typename T1, typename T2>
auto mul(T1 a, T2 b) -> decltype(a * b)
{
    return a * b;
}

int main()
{
    auto v = mul(1.0, 1);
    auto u = mul(1, 1);
}
```


[Go's declaration syntax](https://go.dev/blog/declaration-syntax)  
[The ``Clockwise/Spiral Rule''](https://c-faq.com/decl/spiral.anderson.html)  
