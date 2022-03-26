---
title: Raft笔记1：选举过程
date: 2022-03-07 20:53:01
tags:
- raft
abbrlink: 'raft-election'
categories:
- Concurrency&Go
---
一年前就再看Raft和Paxos，然后想做6.824的Lab，结果当时研一那会一边上课一边做骗钱项目哄客户还要准备毕设开题，6.824的Lab就做了MapReduce和Raft，Raft的那个Lab2有的测试怎么也跑不过，debug捉虫越找越头i。这次想把6.824重新做一遍，索性把原来的代码全删了从头写。
Raft是分布式一致性共识算法Paxos的一种特例
<!-- more -->

# Raft算法简介

Raft是用来解决分布式一致性共识问题的算法，可以看作Paxos的一个特例。起初Lamport老爷子发表Paxos算法的时候是用拜占庭将军问题举例子的，提出了共识问题。老爷子的文章叫做 Paxos Made Simple，一年前本弱智感受到了老爷子深深的嘲讽，这哪里简单了。
Paxos的一个经典实现是chubby，Raft也有etcd之类的工程实现，我们先不去看那些实现，自己照着概念试图实现一个。

![](raft/1646662575.png)

Raft算法通过选举使得一个集群的节点达成一致，实现了最终一致性，节点被分成了三个角色：Leader, Candidate, Follower。节点3种状态的转换是一个状态机。在网络传输的过程中可能会遇到到达缓慢的问题、被攻击/传输过程发生错误，每个节点可能会不可用、会有竞态和票数相同，Raft算法要处理这些问题。看起来简单的三种状态转换的状态就变得麻烦了。

![](raft/1646662617.png)

## 选举过程

选举过程可以看这个动画：[Leader Election](http://thesecretlivesofdata.com/raft/#election)，比较形象。

节点初始都是 Follower，通过选举选出 Leader，我们下面看选举过程。

首先初始状态没有 Leader，进行 Leader 的选举。首先从一群 Followers 中选出candidate


## 6.824 Lab2A

### go实现状态机和选举过程

节点就用一个 `struct` 来表示。状态机的状态转移就用`switch`配合`channel`实现就好了。

## 链接

[Raft Made Simple](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf)  Lamport的文章  
[In Search of an Understandable Consensus Algorithm](https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14) Raft的论文  