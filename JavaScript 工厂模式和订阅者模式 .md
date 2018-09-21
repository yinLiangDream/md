---
title: JavaScript 工厂模式和订阅者模式
tags: JavaScript, 设计模式,代码设计
grammar_cjkRuby: true
---

设计模式的好处：

1. 代码规范

```javascript
// 例如表单验证，两个 input ，一个用户名，一个密码
// 通常做法是
function checkUser() {
  //.....
}
function checkPassword() {
  //.....
}
// 问题：
// 这是两个全局对象，而这两个方法属于一个表单的验证

// 所以这应该是一个表单对象，起码应该这么写
// 对象封装，但是注意全局对象
var checkObj = {
  checkUser: function() {
    //.....
  },
  checkPassword: function() {
    //.....
  }
}(
  // 问题：
  // 对象过多
  // 解决：闭包

  function() {
    //.....
  }
)();
```

## 工厂模式

```javascript
// ajax 为例
var xhr = new XMLHttpRequest(); // IE6不兼容XMLHttpRequest对象，需要使用ActiveXObject("Microsoft.XMLHTTP")
xhr.onreadystatechange = function() {};
xhr.open();
xhr.send(null);
```

```javascript
// 使用工厂模式
// 好处：低耦合，高内聚
var XMLHTTPFactory = function() {
  var XMLHTTP = null;
  if (window.XMLHttpRequest()) {
    XMLHTTP = new XMLHttpRequest();
  } else {
    XMLHTTP = new ActiveXObject('Microsoft.XMLHTTP');
  }
  return XMLHTTP;
};
// 使用
var xhr = XMLHTTPFactory();
```

## 订阅者模式

```javascript
// 订阅者：买家
// 发布者：卖家
var showObj = {}; // 定义发布者
showObj.list = []; // 存储订阅者
// 增加订阅者，订阅消息
showObj.listen = function(fn) {
  // fn回调函数，输出你想输出的内容
  showObj.list.push(fn);
};
showObj.trigger = function() {
  for (var i = 0, fn; (fn = this.list[i++]); ) {
    fn.apply(this, arguments);
  }
};

showObj.listen(function(color, size) {
  console.log('颜色是' + color);
  console.log('尺码是' + size);
});

showObj.listen(function(color, size) {
  console.log('再次输出颜色是' + color);
  console.log('再次输出尺码是' + size);
});

showObj.trigger('白色', 40);
showObj.trigger('黑色', 20);

// 你会发现输出了两遍，这是为什么呢？
// 原因就在于showObj.list.push(fn)
// 那么如何使不同的订阅者查看到属于自己的订阅呢
```

使用一个 key 区分

```javascript
var showObj = {}; // 定义发布者
showObj.list = []; // 存储订阅者
// 增加订阅者，订阅消息
showObj.listen = function(key, fn) {
  // 在订阅的时候给一个标识
  if (!this.list[key]) {
    this.list[key] = [];
  }
  showObj.list[key].push(fn);
};
showObj.trigger = function() {
  // 发布的时候认识知道订阅的消息分别发布给谁
  // 取到消息名称
  var key = Array.prototype.shift.call(arguments);
  // 对应的回调函数
  var fns = this.list[key];
  if (!fns || fns.length === 0) {
    return;
  }
  for (var i = 0, fn; (fn = fns[i++]); ) {
    fn.apply(this, arguments);
  }
};

showObj.listen('white', function(color, size) {
  console.log('小爱同学订阅的颜色是' + color);
  console.log('小爱同学订阅的尺码是' + size);
});

showObj.listen('black', function(color, size) {
  console.log('小冰同学订阅的颜色是' + color);
  console.log('小冰同学订阅的尺码是' + size);
});

showObj.trigger('white', '白色', 40);
showObj.trigger('black', '黑色', 20);
// 但是这样的写的话适用性太差，所以我们需要对其进行封装。
```

封装

```javascript
var event = {
  list: [],
  listen: function(key, fn) {
    // 在订阅的时候给一个标识
    if (!this.list[key]) {
      this.list[key] = [];
    }
    this.list[key].push(fn);
  },
  trigger: function() {
    // 发布的时候认识知道订阅的消息分别发布给谁
    // 取到消息名称
    var key = Array.prototype.shift.call(arguments);
    // 对应的回调函数
    var fns = this.list[key];
    if (!fns || fns.length === 0) {
      return;
    }
    for (var i = 0, fn; (fn = fns[i++]); ) {
      fn.apply(this, arguments);
    }
  },
  // 移除方法（取消订阅）
  remove: function(key, fn) {
    var fns = this.list[key];
    // 没有订阅过
    if (!fns) {
      return false;
    }
    // 回调函数为空
    if (!fn) {
      fn && (fns.length = 0);
    } else {
      for (var i = fns.length - 1; i >= 0; i++) {
        var _fn = fns[i];
        if (_fn === fn) {
          fns.splice(i, 1); //删除订阅者的回调函数
        }
      }
    }
  }
};

var initEvent = function(obj) {
  for (var i in event) {
    obj[i] = event[i];
  }
};
var showObj = {}; // 让任何一个普通对象都拥有发布订阅功能
initEvent(showObj);

showObj.listen(
  'white',
  (fn1 = function(color, size) {
    console.log('小爱同学订阅的颜色是' + color);
    console.log('小爱同学订阅的尺码是' + size);
  })
);

showObj.listen(
  'black',
  (fn2 = function(color, size) {
    console.log('小冰同学订阅的颜色是' + color);
    console.log('小冰同学订阅的尺码是' + size);
  })
);

showObj.trigger('white', '白色', 40);
showObj.trigger('black', '黑色', 20);
```
