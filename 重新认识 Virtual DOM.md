

# 那个争议开端

这件事还要从 2013 年那个秋天说起。



> 这实际上非常快，主要是因为大多数DOM操作往往很慢。DOM上有很多性能工作，但大多数DOM操作都会丢帧。



![2013 React Pete Hunt](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-07-11-065304.jpg)



对！就是这张图，这张图把大家引入了 `DOM` 操作是昂贵且慢的，`Virtual DOM` 是快速的思维里。

6 年后的今天，`React` 已经风靡全球，`Virtual DOM` 也受到了大家的认可，国产之星 `VUE` 也使用了 `Virtual DOM`。

那么问题来了，`Virtual DOM` 真的快吗？`Virtual DOM` 的意义到底是什么？我们为什么要使用 `Virtual DOM`？

我们都听说直接更新文档对象模型（`DOM`）效率低且速度慢。但是，我们中很少有人真的有数据支持它。关于React虚拟DOM的讨论是，它是一种更有效的方式来更新 `Web` 应用程序中的视图，但我们很少有人知道为什么以及这种效率是否会导致更快的页面渲染时间。

抛开使用 `React` 的其他好处，例如单向数据绑定和组件，我将讨论 `Virtual DOM` 到底是什么，以及它是否能够证明 `React` 比其他UI库更合理（或者根本没有UI库） 。



# 我们为什么需要 UI 库

我们为什么需要 UI 库呢？

我敢肯定，现在的前端界，很大一部分人离开了三大框架之后就不知道该怎么办了，他们可能理所当然的认为视图和数据是绑定的（VUE），或者直接使用 `setState` 来更新视图（React）。

有了 `UI` 库之后，我们可以直接数据与视图绑定，而不需要再操作 `DOM`。

# 我们为什么不想操作 DOM

这里不会详细的讲 `DOM`，只会粗略带过。



`DOM`代表*文档对象模型*，是结构化文本的抽象。对于`Web`开发人员，此文本是`HTML`代码，`DOM`简称为*`HTML DOM`*。`HTML`的*元素*成为`DOM`中的*节点*。

`HTML DOM`提供了一个用于遍历和修改节点的接口（API）。它包含像`getElementById`或的方法`removeChild`。我们通常使用`JavaScript`语言来处理`DOM`，因为......好吧，没人知道为什么:)。

因此，每当我们想要动态地更改网页的内容时，我们都会修改`DOM`：

```javascript
var item = document.getElementById("myLI");
item.parentNode.removeChild(item);
```

`document`是根节点的抽象`getElementById`，`parentNode`而且`removeChild`是来自`HTML DOM API`的方法。

那么问题来了，由于`HTML DOM`始终是树形结构，我们可以很容易地遍历每个节点，但是如今`Web APP`的当下，`DOM`树越来越大，我们需要不停的修改大量的`DOM`树。这是真正令人痛苦的地方。

我们通常是以下一个流程来更新 `DOM`：

1. 遍历（或者使用 id）树找到相关的节点
2. 在有必要时更新这个节点

这明显有几个问题：

1. 很难管理。找一个节点并分析上下文的关系，耗时耗力，一不小心喜提`bug`。
2. 效率极低。



# 为什么更新 DOM 很慢

更新`DOM`并不慢，就像更新任何`JavaScript`对象一样。

那究竟是什么让更新真正的`DOM`变慢？

是绘制。

布局过程中，绘制占用了大部分时间。

结合下图，以及此[文章](https://juejin.im/post/5d1492bbe51d4556bc066fb5)，你会明白，更新 `DOM` 的真正问题是屏幕的绘制。

![img](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-07-11-085457.png)



负责在浏览器屏幕上显示或呈现网页的渲染引擎解析`HTML`页面以创建`DOM`。它还解析`CSS`并将`CSS`应用于`HTML`，从而创建渲染树，此过程称为**`attachment`**。

所以，当我们这样做时

```js
document.getElementById('elementId').innerHTML="New Value"
```

发生以下事情：

1. 浏览器必须解析`HTML`
2. 它删除了`elementId` 的子元素
3. 使用"New Value"更新`DOM`
4. 重新计算父和子的`CSS`
5. 更新布局，即每个元素在屏幕上的精确坐标
6. 遍历渲染树并在浏览器显示上绘制它

重新计算CSS和更改布局使用复杂的算法，它们会影响性能。

因此，更新真正的`DOM`并不仅仅涉及更新`DOM`，而是涉及许多其他过程。

此外，上述每个步骤都针对真实`DOM`的每次更新运行，即如果我们更新真实`DOM` 10次，则上述步骤中的每一个将重复10次。这就是为什么更新 `DOM` 很慢的原因。

# 神奇的 Virtual DOM

首先 ， `Virtual DOM`不是由 `React `发明的，但`React`使用它并免费提供。

由于 `DOM` 操作的复杂性，`Virtual DOM`被创造了出来，他以一个虚拟树的状态，存储在内存中，再映射到真实的 `DOM`，每次更新，都是虚拟树的对比，再将差异部分进行更新，并反映到真实 `DOM` 上去，这样我们就减少了对真实 `DOM` 的操作。

在`React`中更新虚拟`DOM`的速度更快，因为`React`使用了

1. 高效的diff算法
2. 批量batching操作
3. 仅有效地更新子树
4. 使用可观察(`observable`)而不是脏检查来检测更改

`AngularJS`使用脏检查来查找已更改的模型。这个脏检查过程在指定时间后循环运行。随着应用程序的增长，检查整个模型会降低性能，从而使应用程序变慢。

每当调用`setState()`方法时，`ReactJS`都会从头开始创建整个`Virtual DOM`。创建整棵树非常快，因此不会影响性能。

在任何给定时间，`ReactJS`维护两个`Virtual DOM`，一个具有更新的状态`Virtual DOM`，另一个具有先前的状态`Virtual DOM`。

使用diff算法比较`Virtual DOM`以找到并更新至`Real DOM`。



让我们举个栗子，首先我们 `state.subject` 的值是 `world`：

```jsx
<div>
  <div id="header">
    <h1>Hello, {{state.subject}}!</h1>
    <p>How are you today?</p>
  </div>
</div>
```



解析后的 `Virtual DOM` 可以表示为：

```jsx
{
  tag: 'div',
  children: [
    {
      tag: 'div',
      attributes: {
        id: 'header'
      },
      children: [
        {
          tag: 'h1',
          children: 'Hello, World!'
        },
        {
          tag: 'p',
          children: 'How are you today?'
        }
      ]
    }
  ]
}
```

现在， `state.subject`的值改变为`Mom`，那么渲染出来的 `Virtual DOM`为：

```jsx
{
  tag: 'div',
  children: [
    {
      tag: 'div',
      attributes: {
        id: 'header'
      },
      children: [
        {
          tag: 'h1',
          children: 'Hello, Mom!'
        },
        {
          tag: 'p',
          children: 'How are you today?'
        }
      ]
    }
  ]
}
```

通过 diff 算法之后，确定只更新的了 `h1` 这个元素，那么将更新的元素再映射到 `DOM` 上即完成了此次的更新。

至于 `batching` 和 `diff 算法` ，内容量较大，需要另开一篇博客讲，目前能搜到的讲解也不少，大家可以去搜搜。

#  Virtual DOM 真的快过直接操作 DOM 吗

关于`Virtual DOM`的每一篇文章和文章都会指出，虽然今天的`JavaScript`引擎速度非常快，但读取和写入浏览器的`DOM`的速度很慢。

这不完全正确。`DOM`很快。添加和删除`DOM`节点并不比在`JavaScript`对象上设置属性慢得多。这只是一个简单的操作。



例如下面一个例子：

这是一个使用原生 `DOM` 渲染的方式：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>Hello JavaScript!</title>

</head>
<body>
<div id="example"></div>
<script>
    document.getElementById("example").innerHTML = "<h1>Hello, world!</h1>";
</script>
</body>
</html>
```

这是一个使用 `React` 实现的方式：

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
    <title>Hello React!</title>
    <script src="build/react.js"></script>
    <script src="build/react-dom.js"></script>
</script>
</head>
<body>
<div id="example"></div>
<script type="text/babel">
    ReactDOM.render(
    <h1>Hello, world!</h1>,
            document.getElementById('example')
    );
</script>
</body>
</html>
```

使用原生需要渲染的时间：

![Load Graph](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-07-11-101549.png)

使用 `React` 需要渲染的时间：

![Load Graph](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-07-11-101554.png)



咋一看，原生渲染速度大大快于 `React`。

但是我们忽略了一个问题，就是页面数据量很少，这样操作，在一个大型列表所有数据都变了的情况下，还算是合理，但是，当只有一行数据发生变化时，它也需要重置整个 `innerHTML`，这时候显然就造成了大量浪费。

比较 `innerHTML` 和 `Virtual DOM` 的重绘过程如下：

- `innerHTML`: render html string ==> 重新创建所有 DOM 元素

- `Virtual DOM`: render Virtual DOM ==> diff ==> 必要的 DOM 更新



和 `DOM` 操作比起来，`js` 计算是非常廉价的。`Virtual DOM render` + `diff` 显然比渲染 `html` 字符串要慢，但是，它依然是纯 `js` 层面的计算，比起后面的 `DOM` 操作来说，依然好了太多。

浏览器在`DOM`更改时必须执行的布局。每次`DOM`更改时，浏览器都需要重新计算`CSS`，进行布局并重新绘制网页，这需要大量时间。

浏览器制造商不断努力缩短重新绘制屏幕所需的时间，可以做的最大的事情是最小化和批量`DOM`更改。

这种减少和批处理`DOM`更改的策略，采用另一个抽象级别，是`React`的`Virtual DOM`背后的理念。



# 最后

`React` 从来没有说过 “`React` 比原生操作 `DOM` 快”。`React`给我们的保证是，在不需要手动优化的情况下，它依然可以给我们提供过得去的性能。

`React`掩盖了底层的 `DOM` 操作，可以用更声明式的方式来描述我们目的，从而让代码更容易维护。

借鉴了知乎上的回答：没有任何框架可以比纯手动的优化 `DOM` 操作更快，因为框架的 `DOM` 操作层需要应对任何上层 `API` 可能产生的操作，它的实现必须是普适的。针对任何一个 `benchmark`，我都可以写出比任何框架更快的手动优化，但是那有什么意义呢？在构建一个实际应用的时候，你难道为每一个地方都去做手动优化吗？出于可维护性的考虑，这显然不可能。