---

---

# 引言

最近面试被问到，`JS` 既然是单线程的，为什么可以执行异步操作？
当时脑子蒙了，思维一直被困在 `单线程` 这个问题上，一直在思考单线程为什么可以额外运行任务，其实在我很早以前写的博客里面有写相关的内容，只不过时间太长给忘了，所以要经常温习啊：[（浅谈 Generator 和 Promise 的原理及实现）](https://juejin.im/post/5b68fec0f265da0f7a1d1f63)

> 1. JS 是单线程的，只有一个主线程
> 2. 函数内的代码从上到下顺序执行，遇到被调用的函数先进入被调用函数执行，待完成后继续执行
> 3. 遇到异步事件，浏览器另开一个线程，主线程继续执行，待结果返回后，执行回调函数

其实 `JS` 这个语言是运行在宿主环境中，比如 `浏览器环境`，`nodeJs环境`

- 在浏览器中，浏览器负责提供这个额外的线程
- 在 `Node` 中，`Node.js` 借助 `libuv` 来作为抽象封装层， 从而屏蔽不同操作系统的差异，`Node`可以借助`libuv`来实现多线程。

而这个异步线程又分为 `微任务` 和 `宏任务`，本篇文章就来探究一下 `JS` 的异步原理以及其事件循环机制

# 为什么 `JavaScript` 是单线程的

`JavaScript` 语言的一大特点就是单线程，也就是说，同一个时间只能做一件事。这样设计的方案主要源于其语言特性，因为 `JavaScript` 是浏览器脚本语言，它可以操纵 `DOM` ，可以渲染动画，可以与用户进行互动，如果是多线程的话，执行顺序无法预知，而且操作以哪个线程为准也是个难题。

所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。

在 `HTML5` 时代，浏览器为了充分发挥 `CPU` 性能优势，允许 `JavaScript` 创建多个线程，但是即使能额外创建线程，这些子线程仍然是受到主线程控制，而且不得操作 `DOM`，类似于开辟一个线程来运算复杂性任务，运算好了通知主线程运算完毕，结果给你，这类似于异步的处理方式，所以本质上并没有改变 `JavaScript` 单线程的本质。

# 函数调用栈与任务队列

## 函数调用栈

`JavaScript` 只有一个主线程和一个调用栈（`call stack`），那什么是调用栈呢？

这类似于一个乒乓球桶，第一个放进去的乒乓球会最后一个拿出来。

举个栗子：

```javascript
function a() {  
  console.log("I'm a!");
};

function b() {  
  a();
  console.log("I'm b!");
};

b();
```



执行过程如下所示：

- 第一步，执行这个文件，此文件会被压入调用栈（例如此文件名为 `main.js`）

  | call stack      |
  | --------------- |
  | <br />`main.js` |

- 第二步，遇到 `b()` 语法，调用 `b()` 方法，此时调用栈会压入此方法进行调用：

  | call stack  |
  | ----------- |
  | <br />`b()` |
  | `main.js`   |

- 第三步：调用 `b()` 函数时，内部调用的 `a()` ，此时 `a()` 将压入调用栈：

  | call stack  |
  | ----------- |
  | <br />`a()` |
  | `b()`       |
  | `main.js`   |

- 第四步：`a()` 调用完毕输出 `I'm a!`，调用栈将 `a()` 弹出，就变成如下：

  | call stack  |
  | ----------- |
  | <br />`b()` |
  | `main.js`   |

- 第五步：`b()`调用完毕输出`I'm b!`，调用栈将 `b()` 弹出，变成如下：

  | call stack      |
  | --------------- |
  | <br />`main.js` |

- 第六步：`main.js` 这个文件执行完毕，调用栈将 `b()` 弹出，变成一个空栈，等待下一个任务执行：

  | call stack |
  | ---------- |
  | <br />     |



这就是一个简单的调用栈，在调用栈中，前一个函数在执行的时候，下面的函数全部需要等待前一个任务执行完毕，才能执行。

但是，有很多任务需要很长时间才能完成，如果一直都在等待的话，调用栈的效率极其低下，这时，`JavaScript` 语言设计者意识到，这些任务主线程根本不需要等待，只要将这些任务挂起，先运算后面的任务，等到执行完毕了，再回头将此任务进行下去，于是就有了 `任务队列` 的概念。



## 任务队列

所有任务可以分成两种，一种是 `同步任务（synchronous）`，另一种是 `异步任务（asynchronous）` 。

同步任务指的是，在主线程上排队执行的任务，只有前一个任务执行完毕，才能执行后一个任务。

异步任务指的是，不进入主线程、而进入`"任务队列"（task queue）`的任务，只有 `"任务队列" `通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。

所以，当在执行过程中遇到一些类似于 `setTimeout` 等异步操作的时候，会交给浏览器的其他模块进行处理，当到达 `setTimeout` 指定的延时执行的时间之后，回调函数会放入到任务队列之中。

当然，一般不同的异步任务的回调函数会放入不同的任务队列之中。等到调用栈中所有任务执行完毕之后，接着去执行任务队列之中的回调函数。

用一张图来表示就是：

![image-20181011232324823](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-11-152327.png)



上图中，调用栈先进行顺序调用，一旦发现异步操作的时候就会交给浏览器内核的其他模块进行处理，对于 `Chrome` 浏览器来说，这个模块就是 `webcore` 模块，上面提到的异步API，`webcore` 分别提供了 `DOM Binding` 、 `network`、`timer` 模块进行处理。等到这些模块处理完这些操作的时候将回调函数放入任务队列中，之后等栈中的任务执行完之后再去执行任务队列之中的回调函数。

我们先来看一个有意思的现象，我运行一段代码，大家觉得输出的顺序是什么：

```javascript
  setTimeout(() => {
    console.log('setTimeout')
  }, 22)
  for (let i = 0; i++ < 2;) {
    i === 1 && console.log('1')
  }
  setTimeout(() => {
    console.log('set2')
  }, 20)
  for (let i = 0; i++ < 100000000;) {
    i === 99999999 && console.log('2')
  }
```

没错！结果很量子化：

![QQ20181012-101019-HD](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-021318.gif)



那么这实际上是一个什么过程呢？那我就拿上面的一个过程解析一下：

- 首先，文件入栈

  ![image-20181012102607896](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-022609.png)

- 开始执行文件，读取到第一行代码，当遇到 `setTimeout` 的时候，执行引擎将其添加到栈中。（由于字体太细我调粗了一点。。。）

  ![image-20181012103026018](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-023027.png)

- 调用栈发现 `setTimeout` 是 `Webapis`中的 `API`，因此将其交给浏览器的 `timer` 模块进行处理，同时处理下一个任务。

![image-20181012134531903](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-054533.png)

- 第二个 `setTimeout` 入栈

  ![image-20181012134755318](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-054757.png)

- 同上所示，异步请求被放入 `异步API` 进行处理，同时进行下一个入栈操作：

  ![image-20181012135048171](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-055049.png)

- 在进行异步的同时，`app.js` 文件调用完毕，弹出调用栈，异步执行完毕后，会将回调函数放入任务队列：

  ![image-20181012140221038](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-060222.png)

- 任务队列通知调用栈，我这边有任务还没有执行，调用栈则会执行任务队列里的任务：

  ![image-20181012140632679](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-060634.png)

  ![image-20181012140723756](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-060725.png)



上面的流程解释了浏览器遇到 `setTimeout` 之后究竟如何执行的，其实总结下来就是以下几点：

1. 调用栈顺序调用任务
2. 当调用栈发现异步任务时，将异步任务交给其他模块处理，自己继续进行下面的调用
3. 异步执行完毕，异步模块将任务推入任务队列，并通知调用栈
4. 调用栈在执行完当前任务后，将执行任务队列里的任务
5. 调用栈执行完任务队列里的任务之后，继续执行其他任务

这一整个流程就叫做 `事件循环（Event Loop）`。



那么，了解了这么多，小伙伴们能从事件循环上面来解析下面代码的输出吗？

```javascript
  for (var i = 0; i < 10; i++) {
    setTimeout(() => {
      console.log(i)
    }, 1000)
  }
  console.log(i)
```



解析：

- 首先由于 `var` 的变量提升，`i` 在全局作用域都有效
- 再次，代码遇到 `setTimeout` 之后，将该函数交给其他模块处理，自己继续执行 `console.log(i)` ，由于变量提升，`i` 已经循环10次，此时 `i` 的值为 `10` ，即，输出 `10`
- 之后，异步模块处理好函数之后，将回调推入任务队列，并通知调用栈
- 1秒之后，调用栈顺序执行回调函数，由于此时 `i` 已经变成 `10` ，即输出10次 `10`

用下图示意：

![image-20181012152514598](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-072516.png)

现在小伙伴们是否已经恍然大悟，从底层了解了为什么这个代码会输出这个内容吧：

![image-20181012152640173](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-072642.png)

那么问题又来了，我们看下面的代码：

```javascript
  setTimeout(() => {
    console.log(4)
  }, 0);
  new Promise((resolve) =>{
    console.log(1);
    for (var i = 0; i < 10000000; i++) {
      i === 9999999 && resolve();
    }
    console.log(2);
  }).then(() => {
    console.log(5);
  });
  console.log(3);
```

大家觉得这个输出是多少呢？

有小伙伴就开始分析了，`promise` 也是异步，先执行里面函数的内容，输出 `1` 和 `2`，然后执行下面的函数，输出 `3` ，但 `Promise` 里面需要循环999万次，`setTimeout` 却是0毫秒执行，`setTimeout` 应该立即推入执行栈， `Promise` 后推入执行栈，结果应该是下图：

![image-20181012161137385](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-12-081139.png)



实际上答案是 `1,2,3,5,4` 噢，这是为什么呢？这就涉及到任务队列的内部，宏任务和微任务。



# 宏任务和微任务

## 什么是宏任务和微任务

任务队列又分为 `macro-task（宏任务）` 与 `micro-task（微任务）` ，在最新标准中，它们被分别称为 `task` 与 `jobs` 。

- `macro-task（宏任务）`大概包括：`script(整体代码)`, `setTimeout`, `setInterval`, `setImmediate（NodeJs）`, `I/O`, `UI rendering`。
- `micro-task（微任务）`大概包括: `process.nextTick（NodeJs）`, `Promise`, `Object.observe(已废弃)`, `MutationObserver(html5新特性)`
- 来自不同任务源的任务会进入到不同的任务队列。其中 `setTimeout` 与 `setInterval` 是同源的。

事实上，事件循环决定了代码的执行顺序，从全局上下文进入函数调用栈开始，直到调用栈清空，然后执行所有的`micro-task（微任务）`，当所有的`micro-task（微任务）`执行完毕之后，再执行`macro-task（宏任务）`，其中一个`macro-task（宏任务）`的任务队列执行完毕（例如`setTimeout` 队列），再次执行所有的`micro-task（微任务）`，一直循环直至执行完毕。 

## 解析

现在我就开始解析上面的代码。

- 第一步，整体代码 `script` 入栈，并执行 `setTimeout` 后，执行 `Promise`：

  ![image-20181013144141327](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-064143.png)

- 第二步，执行时遇到 `Promise` 实例，`Promise` 构造函数中的第一个参数，是在`new`的时候执行，因此不会进入任何其他的队列，而是直接在当前任务直接执行了，而后续的`.then`则会被分发到`micro-task`的`Promise`队列中去。

  ![image-20181013144638756](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-064640.png)

  ![image-20181013144902587](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-064903.png)

- 第三步，调用栈继续执行宏任务 `app.js `，输出`3`并弹出调用栈，`app.js` 执行完毕弹出调用栈：

  ![image-20181013145222565](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-065224.png)

  ![image-20181013145713234](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-065714.png)

- 第四步，这时，`macro-task(宏任务)`中的 `script` 队列执行完毕，事件循环开始执行所有的 `micro-task(微任务)`：

  ![image-20181013150040555](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-070042.png)

- 第五步，调用栈发现所有的 `micro-task(微任务)` 都已经执行完毕，又跑去`macro-task(宏任务)`调用 `setTimeout` 队列：

  ![image-20181013150354612](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-070355.png)

- 第六步，`macro-task(宏任务)` `setTimeout` 队列执行完毕，调用栈又跑去微任务进行查找是否有未执行的微任务，发现没有就跑去宏任务执行下一个队列，发现宏任务也没有队列执行，此次调用结束，输出内容`1,2,3,5,4`。

那么上面这个例子的输出结果就显而易见。大家可以自行尝试体会。

## 总结

1. 不同的任务会放进不同的任务队列之中。
2. 先执行`macro-task`，等到函数调用栈清空之后再执行所有在队列之中的`micro-task`。
3. 等到所有`micro-task`执行完之后再从`macro-task`中的一个任务队列开始执行，就这样一直循环。
4. 宏任务和微任务的队列执行顺序排列如下：
5. `macro-task（宏任务）`：`script(整体代码)`, `setTimeout`, `setInterval`, `setImmediate（NodeJs）`, `I/O`, `UI rendering`。
6. `micro-task（微任务）`: `process.nextTick（NodeJs）`, `Promise`, `Object.observe(已废弃)`, `MutationObserver(html5新特性)`



# 进阶举例

那么，我再来一些有意思一点的代码：

```html
<script>
  setTimeout(() => {
    console.log(4)
  }, 0);
  new Promise((resolve) => {
    console.log(1);
    for (var i = 0; i < 10000000; i++) {
      i === 9999999 && resolve();
    }
    console.log(2);
  }).then(() => {
    console.log(5);
  });
  console.log(3);
</script>
<script>
  console.log(6)
  new Promise((resolve) => {
    resolve()
  }).then(() => {
    console.log(7);
  });
</script>
```

这一段代码输出的顺序是什么呢？

其实，看明白上面流程的同学应该知道整个流程，为了防止一些同学不明白，我再简单分析一下：

- 首先，`script1` 进入任务队列（为了方便起见，我把两块`script` 命名为`script1`，`script2`）：

  ![image-20181013152218883](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-072219.png)

- 第二步，`script1` 进行调用并弹出调用栈：

  ![image-20181013152506563](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-072507.png) 

- 第三步，`script1`执行完毕，调用栈清空后，直接调取所有微任务：

  ![image-20181013152844991](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-072846.png)

- 第四步，所有微任务执行完毕之后，调用栈会继续调用宏任务队列：

  ![image-20181013153031374](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-073033.png)

- 第五步，执行 `script2`，并弹出：

  ![image-20181013153426912](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-073428.png)

- 第六步，调用栈开始执行微任务：

  ![image-20181013153503105](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-073504.png)

- 第七步，调用栈调用完所有微任务，又跑去执行宏任务：

  ![image-20181013153654938](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-073656.png)



至此，所有任务执行完毕，输出 `1,2,3,5,6,7,4`

了解了上面的内容，我觉得再复杂一点异步调用关系你也能搞定：

```javascript
setImmediate(() => {
    console.log(1);
},0);
setTimeout(() => {
    console.log(2);
},0);
new Promise((resolve) => {
    console.log(3);
    resolve();
    console.log(4);
}).then(() => {
    console.log(5);
});
console.log(6);
process.nextTick(()=> {
    console.log(7);
});
console.log(8);
//输出结果是3 4 6 8 7 5 2 1
```

![image-20181013163225154](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-10-13-083226.png)

# 终极测试

```javascript
setTimeout(() => {
    console.log('to1');
    process.nextTick(() => {
        console.log('to1_nT');
    })
    new Promise((resolve) => {
        console.log('to1_p');
        setTimeout(() => {
          console.log('to1_p_to')
        })
        resolve();
    }).then(() => {
        console.log('to1_then')
    })
})

setImmediate(() => {
    console.log('imm1');
    process.nextTick(() => {
        console.log('imm1_nT');
    })
    new Promise((resolve) => {
        console.log('imm1_p');
        resolve();
    }).then(() => {
        console.log('imm1_then')
    })
})

process.nextTick(() => {
    console.log('nT1');
})
new Promise((resolve) => {
    console.log('p1');
    resolve();
}).then(() => {
    console.log('then1')
})

setTimeout(() => {
    console.log('to2');
    process.nextTick(() => {
        console.log('to2_nT');
    })
    new Promise((resolve) => {
        console.log('to2_p');
        resolve();
    }).then(() => {
        console.log('to2_then')
    })
})

process.nextTick(() => {
    console.log('nT2');
})

new Promise((resolve) => {
    console.log('p2');
    resolve();
}).then(() => {
    console.log('then2')
})

setImmediate(() => {
    console.log('imm2');
    process.nextTick(() => {
        console.log('imm2_nT');
    })
    new Promise((resolve) => {
        console.log('imm2_p');
        resolve();
    }).then(() => {
        console.log('imm2_then')
    })
})
// 输出结果是：？
```

大家可以在评论里留言结果哟~