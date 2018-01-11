新事物也是有两面性的，箭头函数有他的便捷有他的优点，但是他也有缺点，他的优点是代码简洁，`this`提前定义，但他的缺点也是这些，比如代码太过简洁，导致不好阅读，`this`提前定义，导致无法使用`JS`进行一些`ES5`里面看起来非常正常的操作。

本质来说箭头函数没有自己的`this`，它的`this`是派生而来的，根据“词法作用域”派生而来。

由于箭头函数在调用时不会生成自身作用域下的`this`和`arguments`值，其持有外部包含它的函数的`this`值，并且在调用的时候就定下来了，不可动态改变，下面我就总结一下什么情况下不该使用箭头函数。

# 在对象上定义函数

```javascript
const test = {
    array: [1, 2, 3],
    sum: () => {
        console.log(this === window); // => true
        return this.array.reduce((result, item) => result + item);
    }
};
test.sum();
// TypeError: Cannot read property 'reduce' of undefined
```
原因就是，箭头函数没有它自己的`this`值，箭头函数内的`this`值继承自外围作用域。

对象方法内的`this`指向调用这个方法的对象，如果使用箭头函数，`this`和对象方法在调用的时候所处环境的`this`值一致。因为 `test.sum()`是在全局环境下进行调用，此时`this`指向全局。

解决方法也很简单，使用函数表达式或者方法简写（ES6 中已经支持）来定义方法，这样能确保 this 是在运行时是由包含它的上下文决定的。


```javascript
const test = {
    array: [1, 2, 3],
    sum() {
        console.log(this === test); // => true
        return this.array.reduce((result, item) => result + item);
    }
};
test.sum();
// 6
```

# 定义原型方法

在对象原型上定义函数也是遵循着一样的规则

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.sayName = () => {
    console.log(this === window); // => true
    return this.name;
};

const cat = new Person('Mew');
cat.sayName(); // => undefined
```
使用传统的函数表达式就能解决问题


```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.sayName = function() {
    console.log(this === Person); // => true
    return this.name;
};

const cat = new Person('Mew');
cat.sayName(); // => Mew
```
# 定义事件回调函数

`this`是`JS`中非常强大的特点，他让函数可以根据其调用方式动态的改变上下文，然后箭头函数直接在声明时就绑定了`this`对象，所以不再是动态的。

在客户端，在`DOM`元素上绑定事件监听函数是非常普遍的行为，在`DOM`事件被触发时，回调函数中的`this`指向该`DOM`，但是，箭头函数在声明的时候就绑定了执行上下文，要动态改变上下文是不可能的，在需要动态上下文的时候它的弊端就凸显出来:


```javascript
const button = document.getElementById('myButton');
button.addEventListener('click', () => {
    console.log(this === window); // => true
    this.innerHTML = 'Clicked button';
});
```
因为这个回调的箭头函数是在全局上下文中被定义的，所以他的`this`是`window`。换句话说就是，箭头函数预定义的上下文是不能被修改的，这样 `this.innerHTML` 就等价于 `window.innerHTML`，而后者是没有任何意义的。

使用函数表达式就可以在运行时动态的改变 `this`：

```javascript
const button = document.getElementById('myButton');
button.addEventListener('click', function() {
    console.log(this === button); // => true
    this.innerHTML = 'Clicked button';
});
```
# 定义构造函数

如果使用箭头函数会报错。

显然，箭头函数是不能用来做构造函数。


```javascript
const Message = (text) => {
    this.text = text;
};
const helloMessage = new Message('Hello World!');
// Throws "TypeError: Message is not a constructor"
```
理论上来说也是不能这么做的，因为箭头函数在创建时this对象就绑定了，更不会指向对象实例。

# 太简短的（难以理解）函数

箭头函数可以让语句写的非常的简洁，但是一个真实的项目，一般由多个开发者共同协作完成，箭头函数有时候并不会让人很好的理解：

```javascript
const multiply = (a, b) => b === undefined ? b => a * b : a * b;
const double = multiply(2);
double(3); // => 6
multiply(2, 3); // => 6
```
代码看起来很简短，但大多数人第一眼看上去可能无法立即搞清楚它干了什么。

这个函数的作用就是当只有一个参数a时，返回接受一个参数b返回a*b的函数，接收两个参数时直接返回乘积。

为了让这个函数更好的让人理解，我们可以为这个箭头函数加一对花括号，并加上return语句，或者直接使用函数表达式：

```javascript
function multiply(a, b) {
    if (b === undefined) {
        return function(b) {
            return a * b;
        }
    }
    return a * b;
}

const double = multiply(2);
double(3); // => 6
multiply(2, 3); // => 6
```
毫无疑问，箭头函数带来了很多便利。恰当的使用箭头函数可以让我们避免使用早期的`.bind()`函数或者需要固定上下文的地方并且让代码更加简洁。

箭头函数也有一些不便利的地方。我们在需要动态上下文的地方不能使用箭头函数：定义对象方法、定义原型方法、定义构造函数、定义事件回调函数。在其他情况下，请尽情的使用箭头函数。
