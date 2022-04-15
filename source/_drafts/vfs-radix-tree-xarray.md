---
title: VFS笔记：Radix-Tree到Xarray的演进
date: 2022-04-09 21:00:01
tags:
- linux
- vfs
- xarray
- radix-tree
abbrlink: 'vfs-radix-tree-xarray'
categories:
- Linux&Unix
---
一直想写点文件系统的相关的东西，之前做6.S081的文件系统Lab也做了。但是每次看到Btrfs和ZFS这些现代文件系统，比较复杂我不懂还是算了，写点ext2什么的又没啥意思。上次写了Reflink，这次还是先写点VFS相关的东西吧，看到对于Xarray的中文介绍比较少，说说这个吧。
<!-- more -->

## VFS简介

>The Virtual File System (also known as the Virtual Filesystem Switch) is the software layer in the kernel that provides the filesystem interface to userspace programs. It also provides an abstraction within the kernel which allows different filesystem implementations to coexist.

VFS（Virtual Filesystem）是为了提供一套统一的文件系统接口，在文件系统上又加了一个抽象层。这样就不用每次换文件系统（Btrfs、ext4、f2fs）或者存储介质（flash、SDD、HDD）的时候换一套API了。另一个好处是兼容不同的介质和文件系统，这个分区是ext4，那个分区是Btrfs，我们可以通过VFS跨文件系统进行移动。

### 基础组件

####  dcache

目录项缓存（Directory Entry Cache， dcache）的相关代码在`fs/dcache.c`，是用来做目录项缓存的，这样就不用

目录项dentry是一个结构体，目录项也可以视为一个文件，一样也有`d_name`, `d_count`(引用计数)

```c
struct dentry {
	/* RCU lookup touched fields */
	unsigned int d_flags;		/* protected by d_lock */
	seqcount_spinlock_t d_seq;	/* per dentry seqlock */
	struct hlist_bl_node d_hash;	/* lookup hash list */
	struct dentry *d_parent;	/* parent directory */
	struct qstr d_name;
	struct inode *d_inode;		/* Where the name belongs to - NULL is
					 * negative */
	unsigned char d_iname[DNAME_INLINE_LEN];	/* small names */

	/* Ref lookup also touches following */
	struct lockref d_lockref;	/* per-dentry lock and refcount */
	const struct dentry_operations *d_op;
	struct super_block *d_sb;	/* The root of the dentry tree */
	unsigned long d_time;		/* used by d_revalidate */
	void *d_fsdata;			/* fs-specific data */

	union {
		struct list_head d_lru;		/* LRU list */
		wait_queue_head_t *d_wait;	/* in-lookup ones only */
	};
	struct list_head d_child;	/* child of parent list */
	struct list_head d_subdirs;	/* our children */
	/*
	 * d_alias and d_rcu can share memory
	 */
	union {
		struct hlist_node d_alias;	/* inode alias list */
		struct hlist_bl_node d_in_lookup_hash;	/* only for in-lookup ones */
	 	struct rcu_head d_rcu;
	} d_u;
} __randomize_layout;
```

dentry有4种状态：
free： 
unused：引用计数为0
in use：引用计数大于0
negative：d_inode为null，这个主要是有的时候经常要查找一个空路径，所以有这么一个negative状态用来缓存

#### superblock object

每种文件系统有一个超级块

#### inode object

#### file object



## 经典的Radix-Tree

基数树Radix-Tree就是前缀树（Trie树）的工程优化变种。前缀树很简单，不过应用不是很多，写业务的时候一般前缀树的活都被map抢了。不过像文件树这些比较稀疏的分层的东西，用树形结构存储正合适。其他的比如搜索的时候搜索建议啊也会用字典树来实现，还有比较经典的IP Route，Radix-Tree用作DNS查询。

linux里面Radix-Tree一般存储`unsigned long`，用于做`long`整形的数和指针的映射。（ID Radix）
ID Radix的作用就类似Radix-Tree用作搜索时的关联数组，可以直接通过ID找到对应的指针。Linux的Radix-Tree和IDR不仅用在VFS里面，还用在设备树等其它很多地方。

```c
struct idr {
        struct radix_tree_root  idr_rt;
        unsigned int            idr_base;
        unsigned int            idr_next;
};
```

以搜索为例简单看一下经典的Radix-Tree。以字符串为例，Trie树的节点保存的是一个char，而Radix-Tree的节点保存的是不相同的一段字符串。这样节省了深度。详情参见：[Radix-Tree](https://en.wikipedia.org/wiki/Radix_tree)  

![](https://upload.wikimedia.org/wikipedia/commons/a/ae/Patricia_trie.svg)


linux的radix-tree实现在`include/linux/radix-tree.h`和`lib/radix-tree.c`
linux内核中的radix-tree实现是加了一个spin_lock。

```c

```


## Xarray

Xarray还是当时在电报群里从fc老师那听到的


```c
struct xarray {
	spinlock_t	xa_lock;
/* private: The rest of the data structure is not to be used directly. */
	gfp_t		xa_flags;
	void __rcu *	xa_head;
};
```


## 链接

[Overview of the Linux Virtual File System](https://www.kernel.org/doc/html/latest/filesystems/vfs.html)  
[Linux lib 之 xarray](https://wushifublog.com/2021/04/16/Linux-lib-%E4%B9%8B-xarray/)   
[[PATCH v8 00/63] XArray v8](https://lkml.iu.edu/hypermail/linux/kernel/1803.0/05298.html)  
[The XArray data structure](https://lwn.net/Articles/745073/) 

[BiscuitOS Radix-Tree](https://biscuitos.github.io/blog/RADIX-TREE/)  
[BiscuitOS IDR](https://biscuitos.github.io/blog/IDR/)
[VFS中的file，dentry和inode](https://bean-li.github.io/vfs-inode-dentry/)  