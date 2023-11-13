---
title: y组合子和不动点
date: 2023-11-09 14:33:23
tags:
- "y-combinator"
abbrlink:
- 'y-combinator'
---
最早接触y组合子的时候，推导是推导出来了，但是y组合子到底有什么用呢。两年前在课上摸鱼看图书馆借来的the little schemer，书里有很多吃的，看着看着就饿了。看到停机问题和y组合子，好像懂了，又好像脑子里全是下课去吃饭。

由于当时忙着下课吃饭鸽了多年的一篇。
<!-- more -->

y-combinator长这样

$$
\lambda f.(\lambda x.f(x \space x)(\lambda x.f(x \space x)))
$$

scheme版本的实现长这样

```lisp
(define Y)
```

蟒蛇版本的长这样：

```python
Y=lambda f: (lambda x: f(x(x)))(lambda x: f(x(x)))
```

y组合子是一种不动点组合子，不动点组合子作用于自身不变。除了y组合子还有

$$
(Y \, g) = (g(Yg))
$$

不动点组合子有什么用呢，不动点组合子可以实现递归。

一般的语言中，我们实现递归只要简单的定义一个函数就可以了，比如scheme版的阶乘（使用define）

```scheme
(define factorial
  (lambda (n)
    (if (= n 0)
      1
      (* n (factorial (- n 1))))))
```

或者c中的阶乘


```c
int factorial(int n) {
    if(n == 0) {
        return 1;
    }
    return factorial(n-1);
}
```

但是在$\lambda$演算中，没有define这个东西（之前说过$\lambda$演算中函数都匿名的）

所以我们要实现define，这就需要用到不动点组合子和一点代换的小技巧。

## 链接

<https://www.slideshare.net/yinwang0/reinventing-the-ycombinator>  
<https://mvanier.livejournal.com/2897.html>  
[Y组合子的一个启发式推导](https://zhuanlan.zhihu.com/p/547191928)  
