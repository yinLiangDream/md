---
title: JQuery 的解读及实现 
tags: JQuery,原型,构造函数
grammar_cjkRuby: true
---

## JQuery的好处
1. 快速上手（学习成本低）
2. 开发效率高（选择器、批量操作 DOM、链型操作……）
3. 一系列的封装（动画、ajax）
4. 浏览器兼容（1.x版本 兼容IE6、7、8）
- JQuery 1.11.3.js（1.x经典版本）
    - 性能不好（源代码文件略大）
- JQuery 2.2.4（2.x经典版本）
    - 性能略好（源代码文件略小）
    - 不是 HTML5 的实现
- 2.x版本对于1.x版本来说，API向下兼容
  - `div.animate({left: 200}, 1000)`在2.2.4版本之后编译不会报错，但是没有动画效果（即为API向下兼容）
  - 2.x版本比1.x版本性能好
  - IE8及以下版本不支持HTML5，所以2.x版本放弃兼容
--- 

## 实现自己的jQuery
### 第一步：分析
```html
<span>this is span</div>
<div class="my-div">this is first div</div>
<div class="my-div">this is second div</div>
```
```javascript
// 第一个问题：想想这句话实际上是执行了什么样的操作？
$(".my-div")
// 解决了第一个问题后，想想如何进行批量操作？
$(".my-div").css("color","red")
$(".my-div").html("--------------")
```
```javascript
// 分析
// 通过原生方法只能执行下面操作
        //通过ID找到唯一元素
document.getElementById()
        //通过标签名找到所有标签名对应的元素
        .getElementsByTagName()
        //通过name属性找到所有对应的元素
        .getElementsByName()
        //是通过selector选择器找到所有符合选择器的元素（方便、快捷、HTML5支持）
        .querySelectorAll()
```

```javascript
// mineJQuery
var $ = function(selector){
    // $选择符已经定义完毕
    return document.querySelectorAll(selector)
    // return 了一个数组[div.my-div, div.my-div]
    // 数组对象没有方法，那.css该如何进行批量操作？
    
    // 除非进行一个 for 循环，接着上面的问题。
    for (var i = 0; i < $(".my-div").length; i++) {
        $(".my-div")[i].style.color = "red";
    }
    // 如果这样操作则使用 jQuery 一点意义都没有 
}
```

### 第二步：构造器函数和普通函数及原型（for循环太多，降低效率）
```javascript
    // 构造器函数是用来通过new关键字创建一个实例（一般构造器函数都是大写开头）
    // 先在池中开辟区间存放变量Proxy
    // 再在堆中开辟区间存放function
    var Proxy = function(selector){
        // this指的是什么呢？谁调用这个函数，指的就是谁！
        // 找到的所有的目标dom元素数组保存在this.doms中
        this.doms = document.querySelectorAll(selector)
    };
    // 函数原型
    Proxy.prototype = {
        css: function(style, value){
            for(var i = 0;i < this.doms.length;i++){
                this.doms[i].style[style] = value;
            }
            // 那么链式操作该怎么进行呢？答案很简单。
            
            // 返回当前实例，目的是让接下来又可以访问Proxy.prototype的方法（即可以进行原型链操作a.b.c.d）
            return this;
        },
        html:function(html){
            for(var i = 0;i < this.doms.length;i++){
                this.doms[i].innerHTML = html;
            }
            return this;
        }
    };
    // 生命周期很长
    // new一个关键字相当于在堆中开辟一个新的空间，都继承了母体（Proxy）的特征，特征全部集中在prototype的原型里
    var p = new Proxy();
    
    var $ = function(selector){
        return new Proxy(selector);
    }
    
    // 基本功能已经实现了，很奈斯！
    // 那么问题又来了，每个原型里面的方法都要进行 for 循环会显得很繁琐，该如何进行优化呢？
```
```javascript
    //PS.----------------------------------------
    // 普通函数
    // 直接在堆中开辟区间执行function（包括了从池中取常量等一系列操作，和闭包相似）
    // 其中各项操作完成后生命周期就结束（内存就被回收）
    var func = function(){
        var a;
        var b = 10;
        a = 20;
    }
    // 调用函数
    func();
```

栈（最小） | 堆（最大） | 池（摆放常量的地方）（中间）
---|---|---
Proxy | function(构造器函数) |0-9、a-z、A-Z、各种符号
X（无）  | function(普通函数)|

### 第三步：改进
- 编程最基本思维：代码复用
```javascript
     Proxy.prototype = {
        // 好处：如果需要修改循环结构，只需要修改each，大大增强代码维护性
        each: function(callback){
            for(var i = 0;i < this.doms.length;i++){
                // call调用：还是调用Proxy函数
                // i => index, this.doms[i] => object
                callback.call(this, i, this.doms[i]);
            }
        },
        css: function(style, value){
            this.each(function(index, object){
                 object.style[style] = value;
                })
            // 返回当前实例，目的是让接下来又可以访问Proxy.prototype的方法（即可以进行原型链操作a.b.c.d）
            return this;
        },
        html:function(html){
             this.each(function(index, object){
                 object.innerHTML = html;
                })
            return this;
        }
    };
    
    // 问题：如果程序员在自己的js文件中写，var Proxy = function(){}
    // 则会与我们写的jQuery方法名冲突
    
    // 原因：是我们写的jQuery里面Proxy是一个全局变量
    // 那该如何进行改进呢？
```

### 第四步：继续改进
```javascript
    // 创建闭包
var $ = jQuery = (function(){
     var Proxy = function(selector){
        this.doms = document.querySelectorAll(selector)
     };
      Proxy.prototype = {
        // 好处：如果需要修改循环结构，只需要修改each，大大增强代码维护性
        each: function(callback){
            for(var i = 0;i < this.doms.length;i++){
            // call调用：还是调用Proxy函数
                callback.call(this, i, this.doms[i]);
            }
        },
        css: function(style, value){
            this.each(function(index, object){
                 object.style[style] = value;
                })
            // 返回当前实例，mu'di'shi目的是让接下来又可以访问Proxy.prototype的方法（即可以进行原型链操作a.b.c.d）
            return this;
        },
        html:function(html){
             this.each(function(index, object){
                 object.innerHTML = html;
                })
            return this;
        }
    };
    // return 出去这个函数
    return function(selector){
        return new Proxy(selector);
    }
})();
// 是不是看起来有模有样很完美了？
// 还是会出现问题：console.log($(..)) ==> Proxy {doms: NodeList[..]}是一个对象，并不是一个数组对象，和原生 jQuery 不一样。
// 那有什么影响呢？
// $(..).length 是属于数组的方法，而我们模仿的 jQuery 是个对象，并没有这个属性。
// 那该如何改进呢？
```
### 第五步：继续改进
```javascript
// 有人会说我直接在 Proxy 方法里加一个 length 就可以了
    var Proxy = function(selector){
        this.doms = document.querySelectorAll(selector);
        this.length = this.doms.length;
    };
// 这样做确实能达到效果，但是有个问题，大家先想想有什么问题。


// 问题就是，jQuery 返回的是个数组对象，但是数组对象不可能只有一个 length 属性，还有很多其他功能，难道我们要把所有数组的功能都加进来吗，明显是不现实的。
// 所以代码需要重新重构一下
```

```javascript
var $ = JQuery = (function(){
    // 创建一个数组
    function markArray(selector) {
        var arr = document.querySelectorAll(selector);
        return arr;
        // 此时返回的确实是一个数组对象，但是感觉一夜回到了解放前，对比看下和真正的 jQuery 的数组对象有什么差别？
    }
    return function(selector) {
        return markArray(selector);
    }
})()

```
对嘛！缺什么补什么嘛！

```javascript
var $ = JQuery = (function(){
    // 创建一个数组
    function markArray(selector) {
        var arr = document.querySelectorAll(selector);
        // console.log(arr.constructor) 看看是个啥
        
        
        // 对 native code 那么 native code 是什么呢？
        // 接地气点讲就是系统自带的本地实现，还说得直接一点就是调用系统本地的 C 源代码库，类似 java 里面 JNI 调用 系统的 C 语言
        
        // 继续往下讲，这个时候我们就能拿到 NodeList ，相当于构造器类
        // 也就是说我们写上 NodeList.prototype.html = function(html){alert(html)}
        // 也就是在 arr 的原型上绑定了这个方法，这个时候就能生效了！
        // 所以，我们要学会在原生的代码里扩展功能
        return arr;
    }
    NodeList.prototype.each = function(callback) {
        // 这个时候可以直接用 this.length，想想为什么？
        for (var i = 0; i < this.length; i++) {
            callback.call(this, i ,this[i])
        }
    }
    NodeList.prototype.html = function(html) {
            this.each(function(index, object) {
                object.innerHTML = html;
            })
    }
    return function(selector) {
        return markArray(selector);
    }
})()

// 改造大功告成，完美运行，可能有人有疑惑了，为啥不用简单方法写呢，例子如下
```

```javascript
// 用眼看不如实际操作一下，你会发现控制台报错了！
var $ = JQuery = (function(){
    // 创建一个数组
    function markArray(selector) {
        var arr = document.querySelectorAll(selector);
        return arr;
    }
    NodeList.prototype = {
        each: function(callback) { 
            for (var i = 0; i < this.length; i++) {
                callback.call(this, i ,this[i])
            }
        },
        html: function(callback) {
            this.each(function(index, object) {
                object.innerHTML = html;
            })
        }
    }
    return function(selector) {
        return markArray(selector);
    }
})()

// 大家注意一下，本身这个思路是没问题的，问题还是出现在 NodeList 是一个 native code ，native code 的原型是不允许被变更的，而上面的操作直接强制改变了 NodeList 的原型，所以只能增加。
// 那么有什么办法呢？
```

### 注意了，下面的写法可能颠覆你的观点
```javascript
// 这个架构和之前的架构有本质上的区别
var $ = JQuery = (function(){
    function markArray(selector) {
        var arr = document.querySelectorAll(selector);
        return arr;
    }
    // 第一步：定义一个局部变量
    var $ = function (selector) {
        return markArray(selector)
    }
    // 自己实现继承
    $.extend = function(target) {
        // 如何进行扩展呢？
        // 我们需要用到 for 循环
        // 为什么要从1开始？
        // 因为 在函数内部有一个 arguments 对象， arguments 这个参数代表的是，调用这个函数所有的参数，即 NodeList 和扩展内容，我们只需要遍历扩展内容
        for (var i = 1; i < arguments.length; i++) {
            // 再次循环遍历出扩展的内容
            for(var prop in arguments[i]) {
                // 一个个加到这个里面来
                target[prop] = arguments[i][prop]
            }
        }
        // ruturn 给了$.fn
        return target;
    }
    // 第二步：基于 NodeList 进行扩展，将后面所有的对象全部扩展到这个里面去，这个思路要好好捋一捋
    // 直接说了，这个代码的意思就是，我调用了一个继承的方法，这个继承的方法在上面自己写，用来传入一些参数，进行扩展 
    // 相当于(target,exd1,exd2,exd3)将 exd1,exd2,exd3的方法扩展给 target
    // 当 target return 出去的时候，这个时候的 target 就是一个超级 NodeList
    // $.fn 的用途以后再说
    $.fn = $.extend(NodeList.prototype, {
        each: function(callback) { 
            for (var i = 0; i < this.length; i++) {
                callback.call(this, i ,this[i])
            }
            // 要想能进行链式操作，所有方法都需要 return this，但其实只需要在 each 方法里 return 就可以了，其他方法只需要将自己的调用 return 出来
            return this;
        },
        html: function(callback) {
            return this.each(function(index, object) {
                object.innerHTML = html;
            })
        }
    })
    return $;
})()
```

```javascript
// jQuery 里有一个插件机制，就是利用$.fn 可以扩展 jQuery 的方法，做到自己定制 jQuery
// 例如 jQuery 自带的和我实现的都触发了这个方法：
$.fn.altr = function(){
    alert(1);
}
$(".my-div").altr();
// 有些人会奇怪了，为什么我没有写这个方法，但是还是能正确触发了这个方法呢？
// 原因在于 $.fn 指向了 NodeList.prototype这个对象，可以在这个之上继续扩展。
// 不信的人可以去试试 console.log($.fn === NodeList.prototype)
// 类似$(function(){})这种的，都是在此类型上继续拓展，进行判断
var $ = function (selector) {
    if (typeof selector === "function") {
        window.onload = selector
    } else {
        return markArray(selector);
    }

}
```