### compose函数

## 什么是compose函数

compose函数是一个工具函数，用于聚合一组函数，依次执行后将前一个函数的执行结果作为参数传入下一个函数，得到最终结果。

例如以下两个函数
```js
    const double = x => x * 2
    const square = x => x * x

```
如果实现以上功能依次调用double，square，可以这样调用

```js
    square(double(5))

```
想通过抽象得到一个函数，能够将任意数量函数依次组合从右至左调用，即是著名的compose函数

```js
    compose(square,double)(5)

```

## 如何实现compose函数

```js
    function compose(...funcs) {
    if (funcs.length === 0) {
        return arg => arg
    }

    if (funcs.length === 1) {
        return funcs[0]
    }

    return funcs.reduce((a, b) => (...args) => a(b(...args)))
    }
```

compose函数的核心代码

```js
    funcs.reduce((a, b) => (...args) => a(b(...args)))
```
funcs是组合函数的数组集合，借助reduce累加器的强大能力，从右至左迭代数组，依次执行左边的函数，并将结果传值右边函数中执行，直至迭代结束拿到最终结果

## compose函数应用

1. 洋葱圈模型

2. redux

3. koa