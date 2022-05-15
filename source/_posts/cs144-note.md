---
title: CS144 Lab0 笔记：环境配置和热身
tags:
  - cs144
  - network
abbrlink: 'cs144-note'
date: 2022-05-07 16:38:17
categories:
- Linux&Unix
---
写习惯了go之后c/cpp水平直线下滑，每三行都有一行忘了加分号。麻了，全靠clang-tidy和报错clang报错救我狗命
<!-- more -->

## 关于cs144

cs144的Lab是实现一个TCP协议栈，可惜是用C++写的，而且那个代码风格我也不怎么喜欢。我C++水平很菜，基本写东西都是C with Class 风格的。而且我不怎么喜欢 Modern C++， 所以不会按照Lab的要求（比如用迭代器代替循环、不使用malloc/free之类的东西、使用智能指针等）

### 做了几个Lab之后的更新

做了三个Lab了回来说一下，这个课程的讲解和Lab的参考材料非常不详细，不像6.S081那样有不少Hints，各种讲解和助教的大作业FAQ什么的也把容易踩坑的地方都提到了。于是CS144的Lab做起来就很麻，颇有一种`fly bitch!`的感觉。

![](cs144-note/1652191785.png)

我做这个东西人麻了的程度甚至比6.824那一堆race条件debug找不出来还要麻。6.824实现Raft好好看论文和资料，大概知道Raft的状态转移和实现思路，剩下的是漫长的debug和race条件抓虫。这个不一样，这个好多思路和细节Lab根本不告诉你，然后实现的时候脑子是懵的，经常不知道干嘛，完全没有思路。然后翻了半天《TCP/IP详解：卷2》，然后各种乱找资料

而且golang我用的比c++，尤其是modern c++熟太多了。然后这个Lab的各种命名也不像xv6那种unix和POSIX风格的精简

## 环境准备

Archlinux下装这些包很简单，直接把[BYO Linux installation](https://stanford.edu/class/cs144/vm_howto/vm-howto-byo.html) 列出来的包用pacman装一遍就好了

需要注意的是NetworkManger要配置一下忽略实验用的虚拟设备。

## Lab 0 

前面几个Lab是用一下telnet和nc玩，跳过（nc用的不多，我还是习惯nmap）

### webget

准备环境：

```bash
git clone https://github.com/cs144/sponge 
cd sponge
mkdir build
cd build
# 我比较喜欢用clang，报错比较舒服
CC=clang CXX=clang++ cmake ..
make -j12
```

这个很简单，直接照着`TCPSocket`的文档改就可以了， 见 <https://cs144.github.io/doc/lab0/class_t_c_p_socket.html>

```cpp
void get_URL(const string &host, const string &path) {
    // Your code here.
    TCPSocket sock1{};
    sock1.connect(Address(host, "http"));
    sock1.write("GET " + path + " HTTP/1.1\r\nHost: " + host + "\r\n\r\n");
    sock1.shutdown(SHUT_WR);
    while (!sock1.eof()) {
        cout << sock1.read();
    }
    sock1.close();
    return;
}
```

然后编译并测试

```bash
make webget
make check_webget
```

### An in-memory reliable byte stream

实现一个内存中有序的字节流。这个算是开胃菜，TCP是在不可靠的网络下实现有序的字节流。这里不要求并发和加锁，所以只要一个非常简单的队列+buffer缓冲就可以了。

>Your byte stream is for use in a single thread—you don’t have to worry about concurrent writers/readers, locking, or race conditions.

看到这句话之前我还以为这个Lab要折腾menory barrier那些东西，这样就好办了。

第一反应是整一个像kqueue那样的循环队列，可是这个Lab的C++风格他不让我用指针，习惯了c风格好难受。算了算了上stl吧。字节流那就直接用队列吧。然后这个类里面再加上是容量（capacity），写入字节，读取字节、是否结束，就差不多了。

```cpp
class ByteStream {
  private:
    // Your code here -- add private members as necessary.
    std::deque<char> _buffer;
    size_t _capacity;
    bool _input_end;
    size_t _byte_written;
    size_t _byte_read;
```

然后我们分别实现这几个类的方法，write好办，注意一下长度就行了。这里要求编码规范是Modern C++, 但是我Modern C++ 不熟，也没怎么用迭代器，蠢蠢的扔了好多for循环之类的东西

write就是一个简单的队尾出队，然后`_byte_written`计数增加

```cpp
size_t ByteStream::write(const string &data) {
    if (_input_end == true) {
        return 0;
    }
    size_t _data_length = data.length();
    if (_data_length + _buffer.size() > _capacity) {
        _data_length = _capacity - _buffer.size();
    }
    _byte_written += _data_length;
    for (size_t i = 0; i < _data_length; i++) {
        _buffer.push_back(data[i]);
    }
    return _data_length;
}
```

peek好办，直接拼成字符串就行了

```cpp
// peek的时候输出是一个string
//! \param[in] len bytes will be copied from the output side of the buffer
string ByteStream::peek_output(const size_t len) const {
    size_t _length = len;
    if (_length > _buffer.size()) {
        _length = _buffer.size();
    }
    string str(_buffer.begin(), _buffer.begin() + _length);
    return str;
}
```

`pop_output`也简单，注意一下长度然后从队列头部pop就好了

```cpp
// buffer里面弹出长度为len的字节，需要考虑len > _buffer.size()的情况
//! \param[in] len bytes will be removed from the output side of the buffer
void ByteStream::pop_output(const size_t len) {
    size_t _data_length = len;
    if (_data_length > _buffer.size()) {
        _data_length = _buffer.size();
    }
    for (size_t i = 0; i < _data_length; i++) {
        _buffer.pop_front();
        _byte_read++;
    }
}
```

然后read就是先`peek_output`再`pop_output`，

```cpp
// 写端的read，注意需要pop_ouput, 读出来之后需要pop的
std::string ByteStream::read(const size_t len) {
    string output = peek_output(len);
    pop_output(len);
    return output;
}
```

然后这一堆方法就直接返回相应的值就好了

```cpp
void ByteStream::end_input() { _input_end = true; }

bool ByteStream::input_ended() const { return _input_end; }

size_t ByteStream::buffer_size() const { return _buffer.size(); }

bool ByteStream::buffer_empty() const { return _buffer.empty(); }

bool ByteStream::eof() const { return _input_end; }

size_t ByteStream::bytes_written() const { return _byte_written; }

size_t ByteStream::bytes_read() const { return _byte_read; }

size_t ByteStream::remaining_capacity() const { return _capacity - _buffer.size(); }
```

emmmmm, 然后`make check_lab0`有两个test没过，看一下报错`Failure message:        The ByteStream should have had eof equal to 0 but instead it was 1`

哦，忘了设置EOF了。不对我设了，那是哪出问题了。。。找了五分钟发现EOF要加上队列empty的条件，我是傻逼

```cpp
bool ByteStream::eof() const { return _input_end && buffer_empty(); }
```

这样测试就过了


## 链接和参考材料

https://cs144.github.io/ 课程主页  
还有视频录像不过我不喜欢看视频，没看，直接上的Lab  
《TCP/IP详解 卷2》 这个有不少实现的细节, Lab讲的不详细我基本上都是跑到这里面翻的  
[Linux 4.4.0内核源码分析——TCP实现](https://github.com/fzyz999/Analysis_TCP_in_Linux) 太感动了这个资料，注释极为详细（不过linux协议栈的实现很复杂，适合没思路的时候偶尔翻翻找找灵感，这个Lab的实现一般都很Naive，差别很大不适合照抄）  

[level-ip](https://github.com/saminiir/level-ip) 一个用户态玩具TCP/IP协议栈，c写的
我最开始先看的这个，对TCP/IP协议栈有个大概的概念，感觉这个看过一遍有了点头绪，CS144的Lab指导看完对整个TCP/IP协议栈是啥样的没啥头绪
[Let's code a TCP/IP stack, 1: Ethernet & ARP](https://www.saminiir.com/lets-code-tcp-ip-stack-1-ethernet-arp/)  
[Let's code a TCP/IP stack, 2: IPv4 & ICMPv4](https://www.saminiir.com/lets-code-tcp-ip-stack-2-ipv4-icmpv4/)   
[Let's code a TCP/IP stack, 3: TCP Basics & Handshake](https://www.saminiir.com/lets-code-tcp-ip-stack-3-tcp-handshake/)  
[Let's code a TCP/IP stack, 4: TCP Data Flow & Socket API](https://www.saminiir.com/lets-code-tcp-ip-stack-4-tcp-data-flow-socket-api/)  
[Let's code a TCP/IP stack, 5: TCP Retransmission](https://www.saminiir.com/lets-code-tcp-ip-stack-5-tcp-retransmission/)  

[rfc.fyi](https://rfc.fyi/) RFC标准速查表  

然后是对写这门课的Lab可能帮助不是特别大的书和资料(资料本身其实都还不错)

《TCP/IP详解 卷1》 这个主要讲概念，看完一遍之后还是不知道怎么实现，如果是为了做Lab找资料推荐直接翻卷2  
《计算机网络: 自顶向下方法》这个书好是好，但是讲的太简略了，看完想实现没啥头绪，还得去TCP/IP协议卷2翻细节，这本书的协议栈实现讲的是`4.4BSD-Lite`，      
《深入理解LINUX网络技术内幕》大量的章节都在讲网络驱动，和这门课的Lab的TCP协议栈的内容关系不是特别大  
《Effective Modern C++ 中文版》 这本书我研一买的，彩色印刷，包装很烂。我当时就想看看modern c++，然后看到一半彻底对C++喜欢不起来了。当时我唯一的想法就是不要把有限的生命浪费在一个充斥着语言律师的语言上。那个时候还在看《The Little Schemer》，看那些优雅连贯的递归的奇妙感觉，和同时在看的c++的繁杂形成了鲜明的对比。也是在那会，被C++坑的多了突然喜欢上go了。  

[Monitoring and Tuning the Linux Networking Stack: Sending Data](https://blog.packagecloud.io/eng/2017/02/06/monitoring-tuning-linux-networking-stack-sending-data)  
[Monitoring and Tuning the Linux Networking Stack: Receiving Data](https://blog.packagecloud.io/eng/2016/06/22/monitoring-tuning-linux-networking-stack-receiving-data/)  
[[译] Linux 网络栈监控和调优：发送数据（2017）](http://arthurchiao.art/blog/tuning-stack-tx-zh/)  
[[译] Linux 网络栈监控和调优：接收数据（2016）](http://arthurchiao.art/blog/tuning-stack-rx-zh/)  这两篇文章适合做完Lab再看，很容易迷失在细节里面  

[A TCP/IP Tutorial](https://www.rfc-editor.org/rfc/rfc1180.txt)  
[[译] RFC 1180：朴素 TCP/IP 教程（1991）](http://arthurchiao.art/blog/rfc1180-a-tcp-ip-tutorial-zh/)   
我一开始还以为是实现一个TCP协议栈的，点开才发现很老的一个简短介绍，大概相当于TCP/IP卷1的太长不看版。其实讲的还可以，没有一点废话，不过对Lab没啥帮助。

[不为人知的网络编程(一)：浅析TCP协议中的疑难杂症(上篇)](http://www.52im.net/thread-1003-1-1.html)
[不为人知的网络编程(二)：浅析TCP协议中的疑难杂症(下篇)](http://www.52im.net/thread-1004-1-1.html) 做Lab4的时候搜资料看到的，解决了我对TCP的一点疑惑  
