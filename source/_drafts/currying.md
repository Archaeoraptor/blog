---
title: 柯里化
date: 2023-11-09 14:00:18
tags:
- currying
abbrlink: 'currying'
---
柯里化（Currying）把接受多个参数的函数变成单个参数的函数
<!-- more -->

比如我们有一个sum函数，接受三个参数

```go
func sum(x, y, z int) int {
    return x + y + z
}
```

改成接受单个变量

```go
func curryingSum(x int) func(int) func(int) int {
    return func(z int) func(int) int {
        return func(y int) int {
            return x + y + z
        }
    }
}
```

于是调用起来就从f(x, y, z)变为f(x)(y)(z)

```go
sum(1, 2, 3)
curryingSum(1)(2)(3)
```

这样就很像管道了，只接受单个参数可以用算子作用上去，比如mapreduce操作的map算子。直觉告诉我们，接收单个变量的函数，比接收多个变量的函数有一些更加良好的性质。

但是go的curringSum函数要写返回值类型，看起来比较难受，用js来一版

```javascript
const curryingAdd = function(x) {
  return function(y) {
    return function(z) {
      return x + y + z
    }
  }
}
```

Curry函数很容易实现

```lisp
(define  (curry n f)  (if (= n 1) f (curry (- n 1) (lambda s (lambda  (a) (apply f (append s (list a))))))))
```

那柯里化有什么用呢，lisp可以少写几个括号（逃

反柯里化