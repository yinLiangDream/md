## Generator

熟悉 ES6 语法的同学们肯定对 Generator（生成器）函数不陌生，这是一个化异步为同步的利器。
栗子：

```javascript
function* abc() {
  let count = 0;
  while (true) {
    let msg = yield ++count;
    console.log(msg);
  }
}

let iter = abc();
console.log(iter.next().value);
// 1
console.log(iter.next('abc').value);
// 'abc'
// 2
```

首先，我们先简单回顾一下 JS 的运行规则：

1. JS 是单线程的，只有一个主线程
2. 函数内的代码从上到下顺序执行，遇到被调用的函数先进入被调用函数执行，待完成后继续执行
3. 遇到异步事件，浏览器另开一个线程，主线程继续执行，待结果返回后，执行回调函数

那么，Generator 函数是如何进行异步化为同步操作的呢？
实质上很简单，\* 和 yield 是一个标识符，在浏览器进行软编译的时候，遇到这两个符号，自动进行了代码转换：

```javascript
// 异步函数
function asy() {
  $.ajax({
    url: 'test.txt',
    dataType: 'text',
    success() {
      console.log('我是异步代码');
    }
  });
}

function* gener() {
  let asy = yield asy();
  yield console.log('我是同步代码');
}
let it = gener().next();
it.then(function() {
  it.next();
});
// 我是异步代码
// 我是同步代码
```

```javascript
// 浏览器编译之后
function gener() {
  // let asy = yield asy(); 替换为
  $.ajax({
    url: 'test.txt',
    dataType: 'text',
    success() {
      console.log('我是异步代码');
      // next 之后执行以下
      console.log('我是同步代码');
    }
  });
  // yield console.log("我是同步代码");
}
```

整个过程类似于，浏览器遇到标识符 \* 之后，就明白这个函数是生成器函数，一旦遇到 yield 标识符，就会将以后的函数放入此异步函数之内，待异步返回结果后再进行执行。

更深一步，从内存上来讲：

普通函数在被调用时，JS 引擎会创建一个栈帧，在里面准备好局部变量、函数参数、临时值、代码执行的位置（也就是说这个函数的第一行对应到代码区里的第几行机器码），在当前栈帧里设置好返回位置，然后将新帧压入栈顶。待函数执行结束后，这个栈帧将被弹出栈然后销毁，返回值会被传给上一个栈帧。

当执行到 yield 语句时，Generator 的栈帧同样会被弹出栈外，但 Generator 在这里耍了个花招——它在堆里保存了栈帧的引用（或拷贝）！这样当 it.next 方法被调用时，JS 引擎便不会重新创建一个栈帧，而是把堆里的栈帧直接入栈。因为栈帧里保存了函数执行所需的全部上下文以及当前执行的位置，所以当这一切都被恢复如初之时，就好像程序从原本暂停的地方继续向前执行了。

而因为每次 yield 和 it.next 都对应一次出栈和入栈，所以可以直接利用已有的栈机制，实现值的传出和传入。

至此，Generator 的魔力已经揭开。

## Promise

Promise 的用法大家应该都很熟悉：

```javascript
let pr = new Promise(function(resolve, reject) {
  setTimeout(function() {
    resolve('成功执行啦');
  }, 2000);
});
pr.then(function(data) {
  console.log(data); // 成功执行啦
});
```

那么 Promise 是如何实现异步加载的呢？

Promise 并没有大家想的那么神秘，其本质就是一个状态机。

想要实现一个土生土长的 Promise 其实很简单，状态机，我们需要几个参数：

- \_\_success_res 用来存储成功时的参数
- \_\_error_res 用来存储失败时的参数
- \_\_status 用来存储状态
- \_\_watchList 用来存储执行队列

下面就手动实现一个 Promise

```javascript
class Promise1 {
  constructor(fn) {
    // 执行队列
    this.__watchList = [];
    // 成功结果
    this.__success_res = null;
    // 失败结果
    this.__error_res = null;
    // 状态
    this.__status = '';
    fn(
      (...args) => {
        // 保存成功数据
        this.__success_res = args;
        // 状态改为成功
        this.__status = 'success';
        // 若为异步则回头执行then成功方法
        this.__watchList.forEach(element => {
          element.fn1(...args);
        });
      },
      (...args) => {
        // 保存失败数据
        this.__error_res = args;
        // 状态改为失败
        this.__status = 'error';
        // 若为异步则回头执行then失败方法
        this.__watchList.forEach(element => {
          element.fn2(...args);
        });
      }
    );
  }

  // then 函数
  then(fn1, fn2) {
    if (this.__status === 'success') {
      fn1(...this.__success_res);
    } else if (this.__status === 'error') {
      fn2(...this.__error_res);
    } else {
      this.__watchList.push({
        fn1,
        fn2
      });
    }
  }
}
```

这样就简单实现了 Promise 的功能，在使用上和 JS 的 Promise 并无其他区别，若想实现 Promise.all 方法，则只需要进行小小的迭代：

```javascript
Promise1.all = function(arr) {
  // 存放结果集
  let result = [];
  return Promise1(function(resolve, reject) {
    let i = 0;
    // 进行迭代执行
    function next() {
      arr[i].then(function(res) {
        // 存放每个方法的返回值
        result.push(res);
        i++;
        // 若全部执行完
        if (i === result.length) {
          // 执行then回调
          resolve(result);
        } else {
          // 继续迭代
          next();
        }
      }, reject);
    }
  });
};
```

至此，Generator 和 Promise 都已解析完成。
