---
title: Go数据类型的并发安全
abbrlink: 'go-container-thread-safety'
tags:
- go
- ring
date: 2022-02-24 10:10:10
categories:
- Concurrency&Go
---
大部分都是不安全的啦，要想并发安全要自己去加锁什么的。
<!-- more -->

## Map

map不是并发安全的的，详见[atomic maps](https://go.dev/doc/faq#atomic_maps)，并发读的时候是安全的，写（upate）的时候是不安全的。不过这个不用担心，不安全的时候会报错的

```go

```

可以自己加读写锁
如果有并发安全需要请使用sync.map（读多写少）
另外就是类似RCU的无锁map（读多写少）

concurrent-map（频繁读），类似分片加锁的思路

## container三件套

这三个当然都不是并发安全的啦

### ring

ring的实现非常简单，不像kfifo那样用了很多技巧。要保证并发安全，除了加锁，更多的做法是自己实现一个lock-free的Ring Buffer

### heap


### list

## 链接

https://github.com/orcaman/concurrent-map  
http://people.cs.pitt.edu/~jacklange/teaching/cs2510-f12/papers/implementing_lock_free.pdf