---
title: 写一个跳表
tags:
  - skiplist
abbrlink: skiplist
categories:
  - 笔记
mathjax: true
date: 2022-05-20 19:14:02
---

写起来确实比树家族简单太多了，树家族的并发我不太会整。为什么要费劲左旋右旋去平衡呢，跳表靠概率分布多么简单优雅啊
<!-- more -->

## 关于跳表

跳表上个世纪的论文里面就有了，不过有业界大量使用貌似比较晚， 貌似是leveldb开始的，后来redis等很多数据库都上了跳表。

跳表在单链表上面加了一层一层的稀疏节点，于是查找的复杂度就降到了$O(\log_2n)$, 类似二分查找，从稀疏节点向下逐级找就行了。如果每一层的节点数量都是下一层的$\frac{1}{2}$

![](skiplist/1653038576.png)

跳表我很喜欢的一个地方是这个玩意不需要像树家族那样去靠左旋右旋来平衡，于是实现就特别简单。

跳表这个结构对缓存不是很友好（链表在内存里面不连续）但是实现很简单，而且加锁比较容易做到加锁力度小。

MuQSS调度器用的就是跳表， 取代了原来BFS调度器里面runqueue的链表，然后查找runqueue的插入复杂度变成了$O(logN)$，然后由于skiplist是有序的，每次只要取出头节点就可以了（优先级最高的），对于MuQSS查找task的复杂度是$O(1)$

MuQSS加锁的时候就直接对每个CPU的skiplist加了一把大锁（task不是很多，加小锁没必要）

这里多说一下BFS调度器的遍历$O(N)$复杂度，这个其实并不慢的，在绝大多数没有跑很多东西的桌面用户，可能链表里面就只有十几个task，遍历起来并不一定比$O(logN)$或者$O(1)$的调度器慢

## 实现

leetcode上就有一道设计跳表的题，直接做呗 [design skiplist](https://leetcode.com/problems/design-skiplist/)

```go
package main

import (
  "fmt"
	"math/rand"
	"time"
)

func main() {
	skl := Constructor()
	skl.Add(1)
	skl.Add(2)
	skl.Add(3)

	fmt.Println(skl.Search(0)) // false
	skl.Add(4)
	fmt.Println(skl.Search(1)) // true
	skl.Add(5)
	skl.Erase(0)
	skl.Erase(1)
	fmt.Println(skl.Search(1)) // true
	// fmt.Println(skl.Search(6)) // false
}

// @lc code=start

const MAX_LEVEL int = 32

type Skiplist struct {
	currentLevel int
	root         *Node
}

type Node struct {
	Val  int
	Prev []*Node
	Next []*Node
}

func init() {
	rand.Seed(time.Now().UnixNano())
}

func randomLevel() int {
	level := 1
	for rand.Intn(2) == 1 {
		level++
	}
	if level > MAX_LEVEL {
		level = MAX_LEVEL
	}
	return level
}

func Constructor() Skiplist {
	var skiplist Skiplist
	var node Node
	skiplist.root = &node
	skiplist.currentLevel = 0
	return skiplist
}

func (sk *Skiplist) Search(target int) bool {
	_, ok := search(sk.root, target)
	return ok
}

func search(root *Node, target int) (node *Node, ok bool) {
	for i := len(root.Next) - 1; i >= 0; i-- {
		var head = root.Next[i]
		if head == nil || head.Val > target {
			continue
		}
		if head.Val == target {
			node, ok = head, true
			return
		}
		if head.Val < target { // 小于则向下
			root = head
			i = len(root.Next)
			continue
		}
	}
	return
}

// 添加元素
func (sk *Skiplist) Add(num int) {
	level := randomLevel() // 随机选择level
	node := Node{
		Val:  num,
		Prev: make([]*Node, level),
		Next: make([]*Node, level),
	}
	add(sk.root, &node)
	if level > sk.currentLevel {
		for i := sk.currentLevel; i < level; i++ {
			sk.root.Next = append(sk.root.Next, &node)
			node.Prev[i] = sk.root
		}
		sk.currentLevel = level
	}
}

func add(root *Node, node *Node) {
	for i := len(root.Next) - 1; i >= 0; i-- {
		var head = root.Next[i]
		if head == nil {
			if i < len(node.Next) {
				root.Next[i] = node
				node.Prev[i] = root
			}
			continue
		}
		if head.Val >= node.Val {
			if i < len(node.Next) {
				root.Next[i] = node
				node.Next[i] = head
				node.Prev[i] = root
				node.Next[i].Prev[i] = node
			}
			continue
		}
		if head.Val < node.Val {
			root = head
			i = len(root.Next)
			continue
		}
	}
}

// 移除元素，然后直接
func (sk *Skiplist) Erase(num int) bool {
	node, ok := search(sk.root, num)
	if !ok {
		return false
	}
	for i := 0; i < len(node.Next); i++ {
		node.Prev[i].Next[i] = node.Next[i]
		if node.Next[i] != nil {
			node.Next[i].Prev[i] = node.Prev[i]
		}
	}
	return true
}
```

## 链接

[Skip Lists: A Probabilistic Alternative to
Balanced Trees](https://15721.courses.cs.cmu.edu/spring2018/papers/08-oltpindexes1/pugh-skiplists-cacm1990.pdf)  
