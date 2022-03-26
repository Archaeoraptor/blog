---
title: Go channel 使用小结
date: 2022-03-05 10:43:22
tags:
- go
- channel
abbrlink: 'go-channel'
categories:
- Concurrency&Go
---
就像面包板元件飞线、PCB布线和拉网线一样，管道（pipeline）是一个很平凡的东西，在Unixlike系统中和go的前世Plan9中随处可见。go的channel当然也像pipeline那样，能对消息进行传递和转移。Channel借鉴的是CSP模型的概念，所以你不能指望用race分析工具去检测死锁，事情没有那么美好……如果再考虑性能问题，哦，麻烦来了……
<!-- more -->

这感觉，用AD画PCB时候布线的痛苦又回来了……满屏goto的感觉又回来了……

## channel通信简介

channel起初是借用CSP模型里的概念，go的goroutine和channel相当于CSP的Process和Channel
但是go并没有完全实现CSP，race检测很多时候都检测不出竞态，用的时候要小心。

>Do not communicate by sharing memory; instead, share memory by communicating.

这里先不说CSP，把Channel当成一个普通的水管。

## 协程安全问题

go的channel使用Mutex锁来保证并发安全

## channel阻塞问题

## 竞态和检测-



## channel的开销

channel不是无锁的(lock free)

## select数量未知的channel

一个不在乎性能的方法是`reflect.Select`

或者可以直接

## channel的实现

channel是用ring buffer

## 链接

[how to listen to N channels? (dynamic select statement)]()


https://www.ulovecode.com/2020/07/14/Go/Golang译文/如何优雅关闭Go-Channel/

[Communicating Sequential Processes](http://www.usingcsp.com/cspbook.pdf)

https://colobu.com/2016/04/14/Golang-Channels/

https://lailin.xyz/post/go-training-week3-channel.html