---
title: 仿Kfifo的go版环形链表
date: 2022-03-11 17:45:39
abbrkink: 'kfifo-like-ring-buffer-in-go'
tags:
- go
- kfifo
---
Kfifo是Linux的一个比较巧妙的环形链表实现，这里模仿一下kfifo的无锁实现。
<!-- more -->

## go container提供的ring

container中的环形链表的实现比较简单，直接看`container/ring/ring.go`的源码就可以了。就是一个首尾相连的双向链表：

```go
type Ring struct {
	next, prev *Ring
	Value      interface{} // for use by client; untouched by this library
}
```

go实现了一个`Do`方法，可以不需要通过`Len`和for循环的方式遍历元素。

```go
    r := ring.New(5)    //新建一个环形链表
	for i := 0; i < 5; i++ {
		r.Value = i
		r = r.Next()
	}
	r.Do(func(v interface{}) {
		fmt.Print(v.(int), " ")
	})
```

除此之外没什么特殊的地方，并不像linux的kfifo的实现那样用了很多技巧。go channel 缓冲区和 k8s 中的一些 ring buffer

## kfifo的实现

我们直接看linux的kfifo代码

```c

```

## 仿kfifo版

