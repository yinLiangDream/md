



上一篇文章我简述了什么是 [Virtual DOM](https://juejin.im/post/5d3ff99fe51d4561fb04beea)，这一章我会详细讲 `Diff` 算法以及为什么在 `React` 和 `Vue` 中循环都需要 key 值。

# 什么是 DOM Diff 算法

Web 界面其实就是一个 DOM 树的结构，当其中某个部分发生变化的时候，实质上就是对应的某个 DOM 节点发生了变化。而在 React/Vue 中，都采用了 Virtual DOM 来模拟真实的树结构，他们都拥有两个 Virtual DOM，一颗是真实 DOM 结构的映射，另一颗则是改动后生成的 Virtual DOM，然后利用高效的 Diff 算法来遍历分析新旧 Virtual DOM 结构的差异，最后 Patch 不同的节点。

但是给定两个 Virtual DOM，利用标准的 Diff 算法肯定是不行的，使用传统的 Diff 算法通过循环递归遍历节点进行对比，其复杂度要达到*O(n^3)*，其中 n 是节点总数，效率十分低下，假设我们要展示 1000 个节点，那么我们就要依次执行上十亿次的比较，这肯定无法满足性能要求。

这里附上一则传统的 Diff 算法：

```js
// 存储比较结果
let result = []
// 比较两棵树
const diffTree = function (beforeTree, afterTree) {
  // 获取较大树的 长度
  let count = Math.max(beforeTree.children.length, afterTree.children.length)
  // 进行循环遍历
  for (let i = 0; i < count; i++) {
    const beforeChildren = beforeTree.children[i]
    const afterChildren = afterTree.children[i]
    // 如果原树没有，新树有，则添加
    if (beforeChildren === undefined) {
      result.push({
        type: 'add',
        element: afterChildren
      })
      // 如果原树有，新树没有，则删除
    } else if (afterChildren === undefined) {
      result.push({
        type: 'remove',
        element: beforeChildren
      })
      // 如果节点名称对应不上，则删除原树节点并添加新树节点
    } else if (beforeChildren.tagName !== afterChildren.tagName) {
      result.push({
        type: 'remove',
        elevation: beforeChildren
      })
      result.push({
        type: 'add',
        element: beforeChildren
      })
      // 如果节点名称一样，但内容改变，则修改原树节点的内容
    } else if (beforeChildren.innerTHML !== afterChildren.innerTHML) {
      // 如果没有其他子节点，则直接改变
      if (beforeChildren.children.length === 0) {
        result.push({
          type: 'changed',
          beforeElement: beforeChildren,
          afterElement: afterChildren,
          html: afterChildren.innerTHML
        });
      } else {
        // 否则进行递归比较
        diffTree(beforeChildren, afterChildren)
      }
    }
  }
  // 最后返回结果
  return result
}
```



然而优化过后的 Diff 算法的复杂度只有*O(n)*，这归结于 DIff 算法的优化，工程师们将 Diff 算法根据实际 DOM 树结构特点做了以下优化。

1. Tree Diff（层级的比较）
2. Component Diff（组件的比较）
3. Element Diff（元素/节点的比较）

简单来说就是两个概念：

- 相同组件会产生类似的 DOM 结构，不同的组件产生的 DOM 结构也不同
- 同一层级的子节点，可以通过唯一的 id 来进行区分



# Tree Diff 层级的比较（树结构的比较）

利用一张常见的图可以完全看出 Tree Diff 的比较规则：

![5518628-d60043dbeddfce8b](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-02-084455.png)

左右两棵树，分别为旧树和新树，先进行树结构的层级比较，并且只会对相同颜色方框内的 DOM 节点进行比较，即同一个父节点下的所有子节点。

如果有任一一个节点不匹配，则该节点和其子节点就会被完全删除，不会继续遍历。

基于这个策略，算法复杂度降低为*O(n)*。

这时有同学要问了，那如果我想移动一个节点到另一个节点下，即跨层级操作，DIff 会怎样表现呢？

如下图所示：

![Xnip2019-08-02_17-21-26](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-03-092824.jpg)

以 C 为根节点，整棵树会被新创建，而不是简单的移动，创建的流程为 `create C`->``create F`->`create G`->`delete C`。

这是一种很影响性能的操作，**官方建议不要进行DOM节点跨层级操作，可以通过CSS隐藏、显示节点，而不是真正地移除、添加DOM节点**。

> 注意：在开发组件时，保持稳定的 DOM 结构会有助于性能的提升。例如，可以通过 CSS 隐藏或显示节点，而不是真的移除或添加 DOM 节点。

# Component Diff 组件比较

`React`/`Vue`是基于组件构建应用的，对于组件间的比较所采用的策略也是非常简洁和高效的。

对此，有以下三种策略：

- 同类型组件（即：两节点是同一个组件类的两个不同实例）
  - 若组件相同，则继续按照层级比较其 Virtual DOM 的结构。
  - 若组件 A 转变为组件 B，但是组件 A 和组件 B 渲染出来的 Virtual DOM 没有任何变化(即，子节点的顺序、状态state等，都未发生变化)，如果开发者能够提前知道这一点，那么可以省下大量 Diff 的时间。React 中，允许用户通过`shouldComponentUpdate()`来判断该组件是否需要进行`diff`算法分析。
- 不同类型组件
  - 直接判断为 `dirty component`，继而替换整个组件的所有内容。

> **注意：**
>
> 1. 如果组件 A 和组件 B 的结构相似，但是 React 判断是 不同类型的组件，则不会比较其结构，而是删除组件 A 及其子节点，创建组件 B 及其子节点。
>
>    举个栗子：就算组件 D 和组件 G 的结构一模一样，但是改变时仍然会删除并且重新创建。
>
>    ![](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-03-092832.png)
>
> 2. Component Diff 只会比较同组节点集合的内容是否改变。即，若旧树里，A 有 B 和 C 两个节点，新树里 A 有 C 和 B 两个节点，无论 B 和 C 的位置是否改变，都会认为 component 层未改变。但是若 A 里的 state 发生了改变，则会认为 component 改变，继而进行组件的更新。

# Element Diff 元素的比较（同一层级同一父元素下的节点集合，进行比较）

当 DOM 处于同一层级时，Diff 提供三个节点操作，即 *删除（REMOVE_NODE）*、*插入（INSERT_MARKUP）*、*移动（MOVE_EXISTING）*。

## 删除（REMOVE_NODE）

旧组件类型，在新集合里也有，但对应的`element`不同则不能直接复用和更新，需要执行删除操作，或者旧组件不在新集合里的，也需要执行删除操作。

如图所示：

![Xnip2019-08-03_13-53-59](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-03-092840.jpg)

## 插入（INSERT_MARKUP）

新的组件类型不在旧集合中，即全新的节点，需要对新节点进行插入操作。

如图所示：

![Xnip2019-08-03_14-00-16](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-03-092844.jpg)

## 移动（MOVE_EXISTING）

旧集合中有新组件类型，且`element`是可更新的类型，这时候就需要做移动操作，可以复用以前的DOM节点。

### 没有 Key 值的问题

如下图，老集合中包含节点：A、B、C、D，更新后的新集合中包含节点：B、A、D、C，此时新老集合进行 diff 差异化对比，发现 B != A，则创建并插入 B 至新集合，删除老集合 A；以此类推，创建并插入 A、D 和 C，删除 B、C 和 D。

![7541670c089b84c59b84e9438e92a8e9_hd](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-03-092853.png)

React 发现这类操作繁琐冗余，因为这些都是相同的节点，但由于位置发生变化，导致需要进行繁杂低效的删除、创建操作，其实只要对这些节点进行位置移动即可。

针对这一现象，React 提出优化策略：允许开发者对同一层级的同组子节点，添加唯一 key 进行区分，虽然只是小小的改动，性能上却发生了翻天覆地的变化！

### 为什么循环需要添加唯一 Key值

给元素加了 Key 值之后，React/Vue 在做 Diff 的时候会进行差异化对比，即通过 key 发现新老集合中的节点都是相同的节点，因此无需进行节点删除和创建，只需要将老集合中节点的位置进行移动，更新为新集合中节点的位置，此时 React 给出的 diff 结果为：B、D 不做任何操作，A、C 进行`移动`操作，即可。

那么，如此高效的 diff 到底是如何运作的呢？

简单来说有以下几步：

1. 对新集合的节点进行遍历，通过唯一 key 可以判断新老集合中是否存在相同节点。

2. 如果存在相同节点，则进行移动操作，但在移动前，需要将当前节点在老集合中的位置与 lastIndex 进行比较，如果不同，则进行节点移动，否则不执行该操作。

   > 这是一种顺序优化手段，lastIndex 一直在更新，表示访问过的节点在老集合中最右的位置（即最大的位置），如果新集合中当前访问的节点比 lastIndex 大，说明当前访问节点在老集合中就比上一个节点位置靠后，则该节点不会影响其他节点的位置，因此不用添加到差异队列中，即不执行移动操作，只有当访问的节点比 lastIndex 小时，才需要进行移动操作。

这里给出一整图作为示例。

![c0aa97d996de5e7f1069e97ca3accfeb_hd](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-03-092900.png)

如上图所示，以新树为循环基准：

1. B 在老集合的下标为 `BIndex=1`，此时 `lastIndex=0`，这时，`lastIndex < BIndex`，不进行任何处理，并且取值 `lastIndex=Math.max(BIndex, lastIndex)`
2. A 在老集合的下标为 `AIndex=0`，此时`lastIndex=1`，这时，`lastIndex > AIndex`，这时，需要把老树中的 A 移动到下标为`lastIndex`的位置，并且取值 `lastIndex=Math.max(AIndex, lastIndex)`
3. D 在老集合的下标为 `DIndex=3`，此时`lastIndex=1`，这时，`lastIndex < DIndex`，不进行任何处理，并且取值 `lastIndex=Math.max(DIndex, lastIndex)`
4. C 在老集合的下标为 `CIndex=2`，此时`lastIndex=3`，这时，`lastIndex > CIndex`，需要把老树中的 C 移动到下标为`lastIndex`的位置，并且取值 `lastIndex=Math.max(CIndex, lastIndex)`
5. 由于 C 已经是最后一个节点，因此 Diff 至此结束。



以上主要分析新老集合中存在相同节点但位置不同时，对节点进行位置移动的情况，如果新集合中有新加入的节点且老集合存在需要删除的节点，那么 React diff 又是如何对比运作的呢？

以此图为例：



![7b9beae0cf0a5bc8c2e82d00c43d1c90_hd](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-03-092905.png)

同上的流程：

1. B 同上流程
2. 老集合中没有 E 集合，则判断老集合中不存在相同节点 E，则创建新节点 E，更新 lastIndex ＝ 1，并将 E 的位置更新为新集合中的位置。
3. C 同上
4. A 同上
5. 当完成集合 Diff 时，最后还需要对老集合进行循环遍历，判断是否存在新集合中没有但老集合中仍然存在的节点，如果有，则删除，循环发现，D 就是这样的节点，因此删除 D，完成 Diff。



这种循环方式，眼尖的读者会发现一个问题，如果是集合的首尾位置互换，那开销就大了。

![1b8dac5b9b3e4452dec8d5447d7717ad_hd](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-03-092910.png)

如上图所示，此时的 DIff 算法，会将 A，B，C 全部移动到 D 的后面，造成大量DOM 的移动，而实际上我们只需要将 D 移动到集合的头部仅一次即可。

> 由此可看出，在开发过程中，尽量减少类似将最后一个节点移动到列表首部的操作，当节点数量过大或更新操作过于频繁时，在一定程度上会影响 React 的渲染性能。



# 没有 key 值的更新问题

除了上述不添加 Key 值会造成整个集合删除再新增，不会进行移动 DOM 操作，导致大量无谓的开销外，但是结合上述 Component Diff 联想，如果 A、B、C、D都是同类型组件且不加 Key 值会发生什么情况呢？

我们看图说话：

这是 Vue 的：

![Jietu20190803-165539-HD](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-03-092940.gif)

这是 React 的：

![Jietu20190803-165152-HD](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-08-03-094900.gif)

我们发现，无论是 React 还是 Vue 删除了第二项之后，第三项列表内部的 state 仍然沿用的第二个列表的内容。

这是因为，React/Vue 判断是变化前后是同类型组件，并且 props 的内容并没有改变，不会触发改变。

其流程如下：

1. 既然 1 没有变，那么就**就地复用**之前的 1
2. 既然 2 变成了 3，里面的子孙元素就地复用。有人不理解为什么子孙元素就地复用，那么是因为子孙元素的 data/state 属性不受 2 变成 3 的影响
3. 既然 3 没了，那么连其子孙元素全部删除

破解方法就是加上唯一的 key，让 Diff 知道就算是同类型的组件，也是有名字区分的。

>在做动态改变的时候，尽量不要使用 index 作为循环的 key，如果你用 index 作为 key，那么在删除第二项的时候，index 就会从 1，2，3 变为 1，2（而不是 1，3），那么仍有可能引起更新错误。



# 总结

- React/Vue 的 DIff 策略使遍历复杂度降低为 O(n)，是一个重大的优化
- React/Vue 在做循环时，一定要加上唯一的 key 值，~~这样不仅能有效提高 Diff 效率~~，减少 DOM 的重绘，还能避免一些稀奇古怪的错误
- 尽量减少跨层级的组件改动，尽量使用 v-show/display:none 来保持 DOM 结构的稳定性，防止新增、删除等消耗大量性能的操作
- 尽量减少将节点尾部移动到节点头部等操作，当节点数量过大或更新操作过于频繁时，在一定程度上会影响 React 的渲染性能。
- 另外，React 从 16 版本开始使用了 Fiber 架构，这个架构解决了大型 React 项目的性能问题及一些之前框架的痛点，我会在下一章详细介绍 Fiber 架构的奥秘和其与之前架构的区别



---

2019-08-05 更新：

经评论区的小伙伴提醒，我发现我写的加 key 能提高 DIff 效率这句话明显出现问题。

>Vue 官方文档：
>
>key 的特殊属性主要用在 Vue 的虚拟 DOM 算法，在新旧 nodes 对比时辨识 VNodes。如果不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试修复/再利用相同类型元素的算法。使用 key，它会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素。
>
>有相同父元素的子元素必须有**独特的 key**。重复的 key 会造成渲染错误。



> React 官方文档：
>
> key 帮助 React 识别哪些项目已更改，已添加或已删除。应该为数组内部的元素赋予键，以使元素具有稳定的标识：key 必须在唯一的。



对于这个问题，我总结出下面几点：

三胞胎站成一排，你怎么知道谁是老大？

如果老大皮了一下子，和老三换了一下位置，你又如何区分出来？

给他们挂个牌牌，写上老大、老二、老三。

这样就不会认错了。key就是这个作用。

说到底，key 的作用就是更新组件时**判断两个节点是否相同**。相同就复用，不相同就删除旧的创建新的。

不加 key 的利弊：

1. 当组件很纯粹，没有数据绑定，基于这个前提下，可以更有效的复用节点，diff 速度来看也是不带 key 更加快速的，因为带 key 在增删节点上有耗时，可以不加 key 以达到就地复用的好处。
2. 这种模式会带来一些隐藏的副作用，比如可能不会产生过渡效果，或者在某些节点有绑定数据（表单）状态，会出现状态错位。

对于 Vue 来说：

- key的作用主要是为了高效的更新虚拟DOM列表,key 值是用来判断 VDOM 元素项的唯一依据。
- 使用key不保证100%比不使用快，这就和Vdom不保证比操作原生DOM快是一样的，这只是一种权衡，其实对于用index作为key是不推荐的，除非你能够保证他们不会发生变化。

对于 React 来说：

- key不是用来提升react的性能的，不过用好key对性能是有帮助的。
- 不能使用random来使用key
- key相同，若组件属性有所变化，则react只更新组件对应的属性；没有变化则不更新。
  key不同，则react先销毁该组件(有状态组件的componentWillUnmount会执行)，然后重新创建该组件（有状态组件的constructor和componentWillUnmount都会执行）



文章最后，如大家有兴趣入小程序的坑，不妨试试用 React 方式书写小程序的框架 Taro，我以此为基础做出一套多端 UI 框架[MP-ColorUI](https://yinliangdream.github.io/mp-colorui-doc/#/)，大家感兴趣可以去 [Github](https://github.com/yinLiangDream/mp-colorui) star 一下，下面是小程序演示版本。

![](https://md-1255362963.cos.ap-chengdu.myqcloud.com/coloruiqrcode.png)