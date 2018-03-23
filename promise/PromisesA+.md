**对于实现者来说，这是一个开放标准，用于实现可靠的，可互操作的JavaScript Promise。**

    promise代表异步操作的最终结果。 与promise互动的主要方式是通过其_then_的方法，该方法注册回调以接收promise的最终值或promise无法执行（fulfill）的原因。
    本规范详细介绍了then方法的行为，所有遵循Promise/A+实现的promise都可以以此为参照基础。本规范应该被认为是非常稳定的。虽然Promises/A+组织可能会偶尔修改此规范，但大都是微小且向下兼容的，主要为了处理一些新发现的边界情况。只有在仔细考虑，讨论和测试之后，我们才会大规模或后向不兼容的更改。
    历史角度说，Promises/A+只是将先前Promises/A提案的行为建议阐明为明确的行为准则，并省略移除未指定或存在问题的内容。
    最后，核心Promises/A+规范不涉及如何创建（create），执行（fulfill）或拒绝（reject）promise，而是选择专注于提供可通用的_then_方法。 未来的配套规范工作可能涉及其他内容。

###1.术语
1.1 “promise”是具有一个其行为符合本规范的_then_方法的对象或函数。
1.2 “thenable”是一个定义_then_方法的对象或函数。
1.3 “value”是任何合法的JavaScript值（包括_undefined_， thenable或promise）。
1.4 “exception”是使用_throw_语句抛出的值。
1.5 “reason”是一个用来表明promise被拒绝的原因的值。

###2.要求
####2.1 promise的状态

promise必须处于以下三种状态中的一种：pending，fulfilled，rejected。

promise满足条件
#####2.1.1 pending状态
    可能转换到fulfilled状态或rejected状态。

#####2.1.2 fulfilled状态
    不能再转换到其他状态。
    必须有一个不能改变的值。

#####2.1.3 rejected状态
    不能再转换到其他状态
    必须有一个不能改变的原因。

在这里，“不能改变”是指不可改变的身份（即===），但并不意味着深层不可变。

####2.2 then方法
    promise必须提供一个_then_方法来获取其当前值或最终值或原因。

    promise的_then_方法接受两个参数：
    promise.then(onFulfilled, onRejected)

#####2.2.1 _onFulfilled_和_onRejected_都是可选参数：
    如果_onFulfilled_不是函数，则必须忽略它。
    如果_onRejected_不是函数，则必须忽略它。

#####2.2.2 如果_onFulfilled_是函数：
    必须在promise状态为fulfilled后被调用，promise的值作为第一个参数。
    在promise状态为fulfilled前不能被调用。
    不能被多次调用。

#####2.2.3 如果_onRejected_是函数：
    必须在promise状态为rejected后被调用，promise的reason是它的第一个参数。
    在promise状态为rejected前不能被调用。
    不能被多次调用。

#####2.2.4 在执行上下文堆栈仅包含平台代码之前，不能调用_onFulfilled_或_onRejected_。注[3.1]

#####2.2.5 _onFulfilled_和_onRejected_必须作为为函数被调用（即没有这个值）。注[3.2]

#####2.2.6 _then_可能会在同一个promise上被多次调用。
    当promise为fulfilled状态时，所有各自的_onFulfilled_回调必须按照其注册的顺序执行_then_。
    当promise为rejected状态时，所有各自的_onRejected_回调必须按照其注册的顺序执行_then_。
#####2.2.7 then必须返回一个promise。注[3.3]
    promise2 = promise1.then(onFulfilled, onRejected);

    如果_onFulfilled_或_onRejected_返回一个值x，则运行下面promise处理程序 [[Resolve]](promise2,x)。
    如果_onFulfilled_或_onRejected_抛出异常e，promise2必须为rejected状态并以e为原因。
    如果_onFulfilled_不是函数并且promise1为fulfilled状态，那么promise2必须为fulfilled状态并与promise1值相同。
    如果_onRejected_不是函数并且promise1为rejected状态，那么promise2必须为rejected状态并与promise1原因相同。

####2.3 promise处理程序
    promise处理程序是一个抽象操作，输入为一个promise和一个值，我们表示为[[Resolve]](promise, x)。如果x是一个thenable且有点像一个promise，处理程序尝试promise采用x的状态。 否则，处理程序用值x执行promise。
    thenable的特性使得promise的实现更通用：只要其暴露出一个遵循Promise/A+的_then_方法即可；这同时也使遵循Promise/A+规范的实现可以与那些不太规范但还可用的实现不相冲突。
    要运行[[Resolve]](promise, x)，请执行以下步骤：

#####2.3.1 如果promise和x引用同一个对象，则以_TypeError_为理由拒绝promise。
#####2.3.2 如果x是一个promise，采用x的状态。注[3.4]
    如果x为pending状态，promise必须保持pending状态直到x为fulfilled或rejected。
    如果x为fulfilled状态，promise为fulfilled状态时与x同样的值
    如果x为rejected状态，promise为fulfilled状态时与x同样的原因
#####2.3.3如果x为一个对象或函数
    把x.then赋给then
    如果检索到属性x.then导致抛出异常e，promise为rejected状态并以e为原因。
    如果then是函数，则使用x作为this调用函数，第一个参数为resolvePromise，第二个参数为rejectPromise，其中：
        当resolvePromise以值y调用，运行[[Resolve]](promise, y)。
        当rejectPromise以原因r被调用时，promise以r为原因拒绝。
        当resolvePromise和rejectPromise都被调用，或者对同一个参数进行多次调用，则第一次调用优先，并且任何进一步调用都会被忽略。
        如果调用_then_抛出异常e
            如果resolvePromise或rejectPromise已被调用，请忽略它。
            否则，promise以e为原因拒绝。
        如果那不是一个函数，promise为fulfilled状态时以x为值。
    如果x不是一个对象或函数，promise为fulfilled状态时以x为值。

如果一个promise是通过参与一个循环的thenable链来解决的，那么[[Resolve]](promise, thenable)的递归性质最终会导致[[Resolve]](promise, thenable)被再次调用， 上述算法将导致无限递归。鼓励实现，但不是必需的，若检测到这种递归，用一个_TypeError_信息来拒绝 promise。注[3.6]
