
所有对象都有__proto__属性，函数这个特殊对象除了具有__proto__属性，还有特有的原型属性prototype。prototype对象默认有两个属性，constructor属性和__proto__属性。prototype属性可以给函数和对象添加可共享（继承）的方法、属性，而__proto__是查找某函数或对象的原型链方式。constructor，这个属性包含了一个指针，指回原构造函数。

所有对象都有隐式原型，即__proto__属性，不是显示原型prototype
每个function 都会有一个prototype属性，这个prototype是显示的，且只有函数对象有

每个function在定义时都会自动创建prototype对象，prototype对象默认带有constructor属性，constructor指向改funtion
当这个函数用于创建新对象，该函数按用途称为构造函数，即用于创建对象的函数，本质上构造函数也是普通函数
因为所有对象都有隐式原型__proto__属性，函数也不例外，函数本身的__proto__指向Function.prototype ,Function.prototype指向Object.prototype再指向null

构造函数创建对象的方法是调用new操作符

构造函数创建对象的方法是调用new操作符
```js
function F(){}      var f = new F()
```
调用new操作符后会返回一个实例对象，该实例对象的隐式原型指向创建该函数的显示prototype
同时构造函数中的this会指向实例对象
```js

            function Parent(name, age) {
                this.name = name || 'noname';
                this.age = age || '20';
                this.sleep = function () {
                    console.log(this.name + '正在睡觉！');
                }
            }

            Parent.prototype.eat = function (food) {
                console.log(`${this.name}正在吃${food}`)
            }
```
原型链继承
```js
function Child() {}
Child.prototype = new Parent();
var p1 = new Child('bob')
```
构造函数继承
```js
            function Child() {
                Parent.call(this);
            }
            var p1 = new Child('bob')
```

组合继承
```js
            function Child() {
                Parent.call(this);
            }
            Child.prototype = new Parent()
            Child.prototype.constructor = Child
            var p1 = new Child('bob')
```

寄生组合继承
```js

            function Child() {
                Parent.call(this)
            }

            function Middle() {}
            Middle.prototype = Parent.prototype

            Child.prototype = new Middle()
            Child.prototype.constructor = Child
            var p1 = new Child('bob')
```


寄生组合继承ES5
```js
            function Child() {
                Parent.call(this)
            }

 	    var middle = Object.create(Parent.prototype)

            Child.prototype = middle 
            Child.prototype.constructor = Child
            var p1 = new Child('bob')
```