# 函数式编程

3/23

## 面向函数编程
从算法角度出发，也就是从行为角度出发，体现的一些编程原则
- 不要重复自己
- 高内聚低耦合
- 最小意外原则
- 单一原则

## 为什么用函数式编程
- 更好的模块化

## 纯函数
> 一个没有任何副作用，而且返回值只由输入决定的函数
> 带有副作用的函数，除了返回值外还会修改某些其它状态，或者与外部函数有可观测的交互

e.g.

slice:从某个已有的数组返回选定的元素。- 纯函数

splic：删除元素，并向数组添加新元素。-带有副作用的函数

```
var x = 5;
function add(y) {return y + x;}
//这种对外部有依赖（X）也不是纯函数

```

```
function pure(x){
  return function(y){
    return x + y
  }
}
pure（5)(1)
//对外部没有依赖每次相同输入对应输出一个相同的结果
```
## 柯里化
> 只传递函数的一部分参数来调用它，让它返回一个函数去处理剩下的参数

```
//非柯里化函数
function pure(x, y) {
  return x + y
}
//柯里化函数
function pure(x){
  return function(y){
    return x + y
  }
}

var r1 = pure(5);
var r2 = r1(6);
var r3 = r1(10);//传一个x后可以多次复用
```
```
  //结合es6
  var add = x => y => x + y

```
> 使用柯里化的好处：

- 参数复用， 利用柯里化我们可以固定住其中部分参数，在调用的时候这个参数就相当于已经被记住了，不需要再次传递
- 延迟执行，不断的柯里化累积传入的参数，最后执行

## 函数的组合 compose

> 当函数纯化之后，有一个特点就是函数可以像堆积木一样组合在一起，变成一个大的函数体
```
let f = x => x + 1;
let h = x => x + 2
let i = x => x + 3;

i(h(f(x))) //函数嵌套
```

```
//简易的 compose函数定义

var compose = function(f, g) {
  return function(x) {
    return f(g(x))
  };
};

//使用 es6 语法

let compose = (f, g) => (x => f(g(x)))

var add x => x + 1
var multiple x => x * 5

var m = compose(add, multiple)(2) //15

```
## 高阶函数

> 以一个函数为参数，同时返回一个函数做为函数的返回值
```
//es6
var hFn = x => y => {
  return y(x)
}
//es5
var hFn = function(x){
  return function(y){
    return y(x)
  }
}
```
## 高阶组件
> 组件也是函数 ，所以也可以效仿高阶函数，把一个组件做为参数传给一个组件然后在组件内部对传进来的组件添加参数，然后返回新的组件
