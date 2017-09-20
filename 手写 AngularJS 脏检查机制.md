# 什么是脏检查

## View -> Model

浏览器提供有`User Event`触发事件的`API`，例如，`click`，`change`等

## Model -> View

浏览器没有数据监测`API`。

`AngularJS` 提供了 `$apply()`，`$digest()`，`$watch()`。

# 其他数据双向绑定介绍

## VUE

`{{}}` `Object.defineProperty()` 中使用 `setter` / `getter` 钩子实现。

## Angular

`[()]` 事件绑定加上属性绑定构成双向绑定。

# 怎么手写

大家先看运行效果，运行后，点增加，数字会+1，点减少，数字会-1，就是这么一个简单的页面，视图到底为何会自动更新数据呢？

我先把最粗糙的源码放出来，大家先看看，有看不懂得地方再议。

老规矩，初始化页面

```html

<!DOCTYPE html>

<html>

<head>

    <meta charset="utf-8">

    <script src="./test_01.js" charset="utf-8"></script>

    <title>手写脏检查</title>

    <style type="text/css">

        button {

            height: 60px;

            width: 100px;

        }

        p {

            margin-left: 20px;

        }

    </style>

</head>

<body>

    <div>

        <button type="button" ng-click="increase">增加</button>

        <button type="button" ng-click="decrease">减少</button> 数量：

        <span ng-bind="data"></span>

    </div>

    <br>

    <!-- 合计 = <span ng-bind="sum"></span> -->

</body>

</html>

```

下面是`JS`源码：`1.0`版本

```javascript

window.onload = function() {

    'use strict';

    var scope = { // 相当于$scope

        "increase": function() {

            this.data++;

        },

        "decrease": function() {

            this.data--;

        },

        data: 0

    }

    function bind() {

        var list = document.querySelectorAll('[ng-click]');

        for (var i = 0, l = list.length; i < l; i++) {

            list[i].onclick = (function(index) {

                return function() {

                    var func = this.getAttribute('ng-click');

                    scope[func](scope);

                    apply();

                }

            })(i)

        }

    }

    function apply() {

        var list = document.querySelectorAll('[ng-bind]');

        for (var i = 0, l = list.length; i < l; i++) {

            var bindData = list[i].getAttribute('ng-bind');

            list[i].innerHTML = scope[bindData];

        }

    }

    bind();

    apply();

}

```

没错，我只是我偷懒实现的……其中还有很多bug，虽然实现了页面效果，但是仍然有很多缺陷，比如方法我直接就定义在了`scope`里面，可以说，这一套代码是我为了实现双向绑定而实现的双向绑定。

回到主题，这段代码中我有用到脏检查吗？

完全没有。

这段代码的意思就是`bind()`方法绑定click事件，`apply()`方法显示到了页面上去而已。

OK，抛开这段代码，先看`2.0`版本的代码

```javascript

window.onload = function() {

    function getNewValue(scope) {

        return scope[this.name];

    }

    function $scope() {

        // AngularJS里，$$表示其为内部私有成员

        this.$$watchList = [];

    }

    // 脏检查监测变化的一个方法

    $scope.prototype.$watch = function(name, getNewValue, listener) {

        var watch = {

            // 标明watch对象

            name: name,

            // 获取watch监测对象的值

            getNewValue: getNewValue,

            // 监听器，值发生改变时的操作

            listener: listener

        };

        this.$$watchList.push(watch);

    }

    $scope.prototype.$digest = function() {

        var list = this.$$watchList;

        for (var i = 0; i < list.length; i++) {

            list[i].listener();

        }

    }

    // 下面是实例化内容

    var scope = new $scope;

    scope.$watch('first', function() {

        console.log("I have got newValue");

    }, function() {

        console.log("I am the listener");

    })

    scope.$watch('second', function() {

        console.log("I have got newValue  =====2");

    }, function() {

        console.log("I am the listener =====2");

    })

    scope.$digest();

}

```

这个版本中，没有数据双向绑定的影子，这只是一个脏检查的原理。

引入2.0版本，看看在控制台发生了什么。

控制台打印出了  `I am the listener` 和 `I am the listener =====2` 这就说明，我们的观察成功了。

不过，仅此而已。

我们光打印出来有用吗？

明显是没有作用的。

接下来要来改写这一段的方法。

首先，我们要使 `listener` 起到观察的作用。

先将 `listener()` 方法输出内容改变，仿照 `AngularJS` 的 `$watch` 方法，只传两个参数：

```javascript

scope.$watch('first', function(newValue, oldValue) {

    console.log("new: " + newValue + "=========" + "old: " + oldValue);

})

scope.$watch('second', function(newValue, oldValue) {

    console.log("new2: " + newValue + "=========" + "old2: " + oldValue);

})

```

再将 `$digest` 方法进行修改

```javascript

$scope.prototype.$digest = function() {

    var list = this.$$watchList;

    for (var i = 0; i < list.length; i++) {

        // 获取watch对应的对象

        var watch = list[i];

        // 获取new和old的值

        var newValue = watch.getNewValue(this);

        var oldValue = watch.last;

        // 进行脏检查

        if (newValue !== oldValue) {

            watch.listener(newValue, oldValue);

            watch.last = newValue;

        }

        // list[i].listener();

    }

}

```

最后将 `getNewValue` 方法绑定到 `$scope` 的原型上，修改 `watch` 方法所传的参数：

```javascript

$scope.prototype.getNewValue = function(scope) {

    return scope[this.name];

}

// 脏检查监测变化的一个方法

$scope.prototype.$watch = function(name, listener) {

    var watch = {

        // 标明watch对象

        name: name,

        // 获取watch监测对象的值

        getNewValue: this.getNewValue,

        // 监听器，值发生改变时的操作

        listener: listener

    };

    this.$$watchList.push(watch);

}

```

最后定义这两个对象：

```javascript

    scope.first = 1;

    scope.second = 2;

```

这个时候再运行一遍代码，会发现控制台输出了 `new: 1=========old: undefined` 和 `new2: 2=========old2: undefined`

OK，代码到这一步，我们实现了watch观察到了新值和老值。

这段代码的 `watch` 我是手动触发的，那个该如何进行自动触发呢？

```javascript

    $scope.prototype.$digest = function() {

        var list = this.$$watchList;

        // 判断是否脏了

        var dirty = true;

        while (dirty) {

            dirty = false;

            for (var i = 0; i < list.length; i++) {

                // 获取watch对应的对象

                var watch = list[i];

                // 获取new和old的值

                var newValue = watch.getNewValue(this);

                var oldValue = watch.last;

                // 关键来了，进行脏检查

                if (newValue !== oldValue) {

                    watch.listener(newValue, oldValue);

                    watch.last = newValue;

                    dirty = true;

                }

                // list[i].listener();

            }

        }

    }

```

那我问一个问题，为什么我要写两个 `watch` 对象？

很简单，如果我在 `first` 中改变了 `second` 的值，在 `second` 中改变了 `first` 的值，这个时候，会出现无限循环调用。

那么，`AngularJS` 是如何避免的呢？

```javascript

    $scope.prototype.$digest = function() {

        var list = this.$$watchList;

        // 判断是否脏了

        var dirty = true;

        // 执行次数限制

        var checkTime = 0;

        while (dirty) {

            dirty = false;

            for (var i = 0; i < list.length; i++) {

                // 获取watch对应的对象

                var watch = list[i];

                // 获取new和old的值

                var newValue = watch.getNewValue(this);

                var oldValue = watch.last;

                // 关键来了，进行脏检查

                if (newValue !== oldValue) {

                    watch.listener(newValue, oldValue);

                    watch.last = newValue;

                    dirty = true;

                }

                // list[i].listener();

            }

            checkTime++;

            if (checkTime > 10 && checkTime) {

                throw new Error("次数过多！")

            }

        }

    }

```

```javascript

scope.$watch('first', function(newValue, oldValue) {

    scope.second++;

    console.log("new: " + newValue + "=========" + "old: " + oldValue);

})

scope.$watch('second', function(newValue, oldValue) {

    scope.first++;

    console.log("new2: " + newValue + "=========" + "old2: " + oldValue);

})

```

这个时候我们查看控制台，发现循环了10次之后，抛出了异常。

这个时候，脏检查机制已经实现，是时候将这个与第一段代码进行合并了，`3.0` 代码横空出世。

```javascript

window.onload = function() {

    'use strict';

    function Scope() {

        this.$$watchList = [];

    }

    Scope.prototype.getNewValue = function() {

        return $scope[this.name];

    }

    Scope.prototype.$watch = function(name, listener) {

        var watch = {

            name: name,

            getNewValue: this.getNewValue,

            listener: listener || function() {}

        };

        this.$$watchList.push(watch);

    }

    Scope.prototype.$digest = function() {

        var dirty = true;

        var checkTimes = 0;

        while (dirty) {

            dirty = this.$$digestOnce();

            checkTimes++;

            if (checkTimes > 10 && dirty) {

                throw new Error("循环过多");

            }

        }

    }

    Scope.prototype.$$digestOnce = function() {

        var dirty;

        var list = this.$$watchList;

        for (var i = 0; i < list.length; i++) {

            var watch = list[i];

            var newValue = watch.getNewValue();

            var oldValue = watch.last;

            if (newValue !== oldValue) {

                watch.listener(newValue, oldValue);

                dirty = true;

            } else {

                dirty = false;

            }

            watch.last = newValue;

        }

        return dirty;

    }

    var $scope = new Scope();

    $scope.sum = 0;

    $scope.data = 0;

    $scope.increase = function() {

        this.data++;

    };

    $scope.decrease = function() {

        this.data--;

    };

    $scope.equal = function() {

    };

    $scope.faciend = 3

    $scope.$watch('data', function(newValue, oldValue) {

        $scope.sum = newValue * $scope.faciend;

        console.log("new: " + newValue + "=========" + "old: " + oldValue);

    });

    function bind() {

        var list = document.querySelectorAll('[ng-click]');

        for (var i = 0, l = list.length; i < l; i++) {

            list[i].onclick = (function(index) {

                return function() {

                    var func = this.getAttribute('ng-click');

                    $scope[func]($scope);

                    $scope.$digest();

                    apply();

                }

            })(i)

        }

    }

    function apply() {

        var list = document.querySelectorAll('[ng-bind]');

        for (var i = 0, l = list.length; i < l; i++) {

            var bindData = list[i].getAttribute('ng-bind');

            list[i].innerHTML = $scope[bindData];

        }

    }

    bind();

    $scope.$digest();

    apply();

}

```

页面上将 合计 放开，看看会有什么变化。

这就是 `AngularJS`脏检查机制的实现，当然，`Angular` 里面肯定比我要复杂的多，但是肯定是基于这个进行功能的增加，比如 `$watch` 传的第三个参数。

# 技术发展

现在 `Angular` 已经发展到了 `Angular5`，但是谷歌仍然在维护 `AngularJS`，而且，并不一定框架越新技术就一定越先进，要看具体的项目是否适合。

比如说目前最火的 `React` ，它采用的是`虚拟DOM`，简单来说就是将页面上的`DOM`和`JS`里面的`虚拟DOM`进行对比，然后将不一样的地方渲染到页面上去，这个思想就是`AngularJS`的脏检查机制，只不过`AngularJS`是检查的数据，`React`是检查的`DOM`而已。