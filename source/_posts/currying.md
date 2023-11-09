---
title: 关于柯里化
tags:
  - currying
abbrlink: currying
categories:
  - 笔记
date: 2023-11-09 14:00:18
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

这样看起来性质很良好，只接受一个参数，返回一个输出。直觉告诉我们，接收单个变量的函数，比接收多个变量的函数有一些更加良好的性质。

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

scheme版本的看起来也跟js差不多

```scheme
(define (currySum f)
    (lambda(x)
        (labmda(y)
            (lambda(z)
                (f x y z)))))
```

对n个变量的Curry函数很容易实现

```lisp
(define  (curry n f)  
 (if (= n 1) f
  (curry 
    (- n 1) 
    (lambda s (lambda  (a) (apply f (append s (list a))))))))
```

js版本：

```javascript
function curry(fn){
    var args = Array.prototype.slice.call(arguments,1);
    return function(){
        var innerArgs = Array.prototype.slice.call(arguments);
        var finalArgs = args.concat(innerArgs);
        return fn.apply(null, finalArgs);
    }
}
```

类似的，我们可以实现反柯里化，说明多变量函数和单变量单返回函数可以相互转换

```scheme
(define ((uncurry f) . t)
  (if (null? t)
      f
      (apply (uncurry (f (car t)))
             (cdr t))))
```

那柯里化有什么用呢，可以用来实现多参数函数（逃

在手搓自制scheme中，我们已经实现了有一个接收一个输入、返回一个输出的函数，那么可以通过柯里化实现一个接收多参数的函数。

## link

[柯里化](https://zh.wikipedia.org/wiki/柯里化)  