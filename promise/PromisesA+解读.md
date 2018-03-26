# promises/A+规范解读

通过Promises/A+规范，可以对于什么是promise，以及promise行为特征有一个全面的认识

## 为什么promise？

promise是一种异步编程模型的具体实现。在现代web工程中，随着工程状态，循环嵌套，异常难以捕获等一系列问题显然成为前端工程代码可读性，可维护性的痛点。在社区的不断摸索下，新的异步编程模型--promise诞生。并在ecmascript2015（即ES6）版本中正式纳入javascript语言规范。

## 怎么用promise？

promise怎么使用？暂且先不做说明，本文旨在从promises/A+规范入手，分析promise的行为，一步步推敲出promise。

## 怎样才是promise？

promise是一个提供了then方法的对象或函数。

### 怎样的对象或函数？

* promise必须有且仅有三个状态pending（等待中），fulfilled（已完成），rejected（已拒绝）。
* promise必须处于这三个状态的其中一种。
* promise处于pending状态时有可能转变成fulfilled状态或rejected状态。
* promise处于fulfilled状态时不能再转换到其他状态，必须有一个不能改变的值。
* promise处于rejected状态时不能再转换到其他状态，必须有一个不能改变的原因。

也就是说promise的特征是拥有三个状态，一旦pending状态结束，promise状态就会转换为fulfilled状态或rejected状态，fulfilled状态或rejected状态成为最终状态并带有对应结果值，固定后不可再改变。

### 怎样的then方法？

* promise必须提供一个then方法来获取其结果值。
* promise的then方法接受两个参数：
```js
 promise.then(onFulfilled, onRejected)
 ```
 * onFulfilled和onRejected是可选参数。
 * 如果onFulfilled和onRejected不是函数，忽略掉。
 * 如果onFulfilled和onRejected是函数，必须在由pending状态转换为最终状态后调用，不能被多次调用。
 * 同一个promise上then方法可以被调用多次，按注册顺序执行。
 * then方法必须返回一个promise。

 可以看出then方法就是一个包含两个可选参数的方法，onFulfilled和onRejected参数分别用来接收fulfilled状态或rejected状态对应的结果值，这两个参数只能在为函数类型时运作，只能在promise转换到最终状态后调用这两个参数方法，且不能被多次调用。同一个promise的then方法可以多次调用，按注册顺序执行。还有最重要的一点，then方法必须要再返回一个promise，下面来具体分析。

 ### 又返回一个promse？

then方法要再返回一个promise.
```js
promise2 = promise1.then(onFulfilled, onRejected);
``` 

* 如果onFulfilled或onRejected返回一个值x，则运行下面promise处理程序 [[Resolve]]。
* 如果onFulfilled或onRejected抛出异常e，promise2必须为rejected状态并以e为原因。
* 如果onFulfilled不是函数并且promise1为fulfilled状态，那么promise2必须为fulfilled状态并与promise1值相同。
* 如果onRejected不是函数并且promise1为rejected状态，那么promise2必须为rejected状态并与promise1原因相同。

then方法的以上几种情况
a.返回一个值，返回前对值得不同情况处理
b.抛出异常时，返回的promise承接
c.then方法参数不是函数，promise1得到最终结果，返回的promise承接

### 怎么样的promise处理程序

```js
[[Resolve]](promise, x)
```

* promise处理程序是对onFulfilled或onRejected返回值x的处理
* promise处理程序输入为一个promise和一个值，如果x是一个类promise（不完全符合promises/A+规范，但含有then方法具有promise的某些特征）
* 如果promise和x引用同一个对象，则以TypeError为理由拒绝promise
* 如果x是一个promise，采用x的状态。
* 如果x为一个对象或函数，把x.then赋给then，如果检索属性x.then导致抛出异常e，promise为rejected状态并以e为原因。
* 如果then是函数，则使用x作为this调用函数，第一个参数为resolvePromise，第二个参数为rejectPromise。
* 如果then不是函数，promise为fulfilled状态时以x为值。
* 如果x不是对象或函数，promise为fulfilled状态时以x为值。

promise处理程序处理then方法的返回值，并将返回值作转换处理，返回promise或将普通值包装为promise返回，遇到异常会向后传递。这种机制为then方法的链式调用和异常捕获提供了基础方案。

## 什么是promise?

致此，通过解读promises/A+规范全面认识了promise核心及行为特性，完成了对promise拆卸，后文会继续解读promise API以及如何按照规范一步一步实现一个promise。
