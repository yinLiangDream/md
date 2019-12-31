> 做前端的同学们肯定或多或少听说过`CSR`，`SSR`，`Prerender`这些名词，但是大多肯定只是停留在听说过，了解过，略懂一点，但是，你真的理解这些技术吗？
>
> 这些名词具体是什么意思呢？
>
> 为什么会产生这种技术，要解决的问题是什么呢？
>
> 每种技术背后的原理又是什么呢？

## 从各自的概念和执行流程说起

在了解这些概念之前，我们要先了解一个熟知的概念，那就是 `SPA(Single Page Application)`，没错，就是大家熟知的单页应用，其实 `CSR、SSR、Prerender` 都是基于 `SPA`，关于 `SPA` 的概念我就不多阐述了。

### CSR(Client Side Render)（客户端渲染）

即，渲染过程全部交给浏览器进行处理，服务器不参与任何渲染。

打包下来页面是这个样子：

```html
<body>
  <noscript>You need to enable JavaScript to run this app.</noscript>
  <div id="root"></div>
</body>
<script src="/static/js/2.22fca29f.chunk.js"></script>
<script src="/static/js/main.a9f5ef89.chunk.js"></script>
```

流程：浏览器 --> 服务器 --> index.html(白屏) --> bundle.js --> images --> Render

![image-20191231113956671](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-12-31-134204.png)

我们来使用 `create-react-app` 来建立一个 `web` 工程，并在 `Chrome` 里使用 `slow 3G` 网络下做个实验：

![Xnip2019-12-29_17-49-12](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-12-31-134215.jpg)

可以看到，从页面空白到元素绘制用了足足 12 秒左右，这个白屏时间太可怕了，这也就是为什么在 `Web App` 盛行的当下，包体积越来越大会导致白屏时间越来越长，大家想要优化这个现象的原因。

### Prerender(Pre Render)（预渲染）

即，打包的时候就预先渲染页面，所以在请求到 `index.html` 就已经是渲染过的内容

流程：浏览器 --> 服务器 --> index.html(预渲染的内容) --> Render --> bundle.js + images --> Render

![image-20191231113759430](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-12-31-134223.png)

我们将刚刚的工程加入 `prerender-spa-plugin` 这个插件，再次运行看看结果

这次打包下来的主页 html 是这个样子的：

```html
<body>
  <noscript>You need to enable JavaScript to run this app.</noscript>
  <div id="root">
    <div class="App">
      <header class="App-header">
        <img
          src="/static/media/logo.5d5d9eef.svg"
          class="App-logo"
          alt="logo"
        />
        <p>Edit <code>src/App.js</code> and save to reload.</p>
        <a
          class="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
          >Learn React</a
        >
      </header>
    </div>
  </div>
</body>
<script src="/static/js/2.22fca29f.chunk.js"></script>
<script src="/static/js/main.a9f5ef89.chunk.js"></script>
```

首页就已经预渲染好了，这时我们再来运行一次看看：

![Xnip2019-12-30_13-54-09](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-12-31-134232.jpg)

此时可以看到，页面只用了 2 秒就已经渲染出元素，不会造成长时间的白屏问题

### SSR(Server Side Render)（服务器渲染）

流程：浏览器 --> 服务器 --> 服务器执行渲染 --> index.html(实时渲染的内容)) --> Render --> bundle.js + images --> Render

![image-20191231111204363](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-12-31-134240.png)

可见 `SSR` 在服务端多做了一些实时渲染的操作，那么我们这次运行下来回事什么结果呢？

![Xnip2019-12-31_11-17-55](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-12-31-134247.jpg)

可以看出来，`SSR` 和 `Prerender` 的效果一致，都能很好的减少白屏时间

### 总结

从上面的实验，可以看出来，无论是 `SSR` 还是 `Prerender`，我们要解决的问题主要是白屏时间太长的问题，这两种技术都是为了解决 `CSR` 的不足之处，那么这两种方案有什么区别？使用场景又有哪些呢？

> P.S 其实另一方面的原因是，`CSR` 对 `SEO` 太不友好了，搜索引擎抓取不到关键信息，只能抓取一个毫无元素的白屏页面，会导致搜索引擎搜索不到你的页面信息进行推荐，`SSR` 和 `Prerender` 都能很好的解决这个问题。（吐槽一下：`Google` 已经实现了抓取基于 `SPA` 的 `CSR`）

## Prerender or SSR

在做出选择之前，我们必须要充分的了解两者的差异。

### Prerender 更加通用但是局限性太大

1. `Prerender` 原理是在构建阶段就将 `html` 页面渲染完毕，不会进行二次渲染，也就是说，当初打包时页面是怎么样，那么预渲染就是什么样，如果页面上有数据实时更新，那么浏览器第一次加载的时候只会渲染当时的数据，等到 JS 下载完毕再次渲染的时候才会更新数据更新，会造成数据延迟的错觉。

2. `Prerender` 需要预先指定需要渲染的页面，需要手动在 webpack 里设置

   ```jsx
   new PrerenderSPAPlugin({
     staticDir: path.join(__dirname, "../", "build"),
     routes: ["/"]
   });
   ```

   所以页面数量很大的情况下，想将每个页面进行预渲染是很大工作量，而且打包时间会很长，还可能会遗漏

说了那么多利弊，那么，预渲染是怎么做到生成页面的呢？

做过爬虫的同学肯定知道 headless 的概念

> Headless Chrome 在 Chrome59 中发布，用于在 headless 环境中运行 Chrome 浏览器，也就是在非 Chrome 环境中运行 Chrome。它将 Chromium 和 Blink 渲染引擎提供的所有现代 Web 平台功能引入命令行。
> 它有什么用处呢？
> headless 浏览器是自动测试和服务器环境的绝佳工具，您不需要可见的 UI shell。例如，针对真实的网页进行测试，创建网页的 PDF，或者只是检查浏览器如何呈现 URL。

`Prerender` 就是利用 `Chrome` 官方出品的 `Puppeteer` 工具，对页面进行爬取。它提供了一系列的 API, 可以在无 UI 的情况下调用 `Chrome` 的功能, 适用于爬虫、自动化处理等各种场景。它很强大，所以很简单就能将运行时的 HTML 打包到文件中。

原理是在 `Webpack` 构建阶段的最后，在本地启动一个 `Puppeteer` 的服务，访问配置了预渲染的路由，然后将 `Puppeteer` 中渲染的页面输出到 `HTML` 文件中，并建立路由对应的目录。

下面是流程图

![image-20191231143313360](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-12-31-134254.png)

### SSR 虽好但是要框架支持

目前来说，主流的框架 `React`、`Vue` 都已经支持 `SSR`，只是配置会繁琐点，有人就会疑惑，框架还要支持 `SSR`？

可事实是，正是因为现代 `SPA` 的 `Virtual DOM` 的存在，才能使 `SSR` 变成现实，但是，`SSR` 这种理念的实现，并非易事。

我们先来看看详细的 `SSR` 流程图:

![image-20191231172059262](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2019-12-31-134259.png)

可以看出，`SSR` 和 `Prerender` 的最大区别就在于，`Prerender` 是静态的，`SSR` 是动态的，`SSR` 会在服务端实时构建出对应的 `DOM`。

这也是 `SSR` 的难点所在：同构（即服务器和浏览器共同构建）。

### 何为同构

同构这个概念存在于 `Vue`，`React` 这些新型的前端框架中，同构实际上是客户端渲染和服务器端渲染的一个整合。我们把页面的展示内容和交互写在一起，让代码执行两次。在服务器端执行一次，用于实现服务器端渲染，在客户端再执行一次，用于接管页面交互。

上面我们说过，`SSR` 的工程中，`React` 代码会在客户端和服务器端各执行一次。你可能会想，这没什么问题，都是 `JavaScript` 代码，既可以在浏览器上运行，又可以在 `Node` 环境下运行。但事实并非如此，如果你的 `React` 代码里，存在直接操作 `DOM` 的代码，那么就无法实现 `SSR` 这种技术了，因为在 `Node` 环境下，是没有 `DOM` 这个概念存在的，所以这些代码在 `Node` 环境下是会报错的。

但是就是由于 `Virtual DOM` 技术的存在，让这一切变成了可能，这里不过多介绍 `Virtual DOM`，简单来说，它就是一个普通的 `JS` 对象，只不过映射了 `HTML DOM` 的结构，`React` 在做页面操作时，实际上不是直接操作 `DOM`，而是操作 `Virtual DOM`，也就是操作普通的 `JavaScript` 对象，这就使得 `SSR` 成为了可能。

我们可以直接在代码里判断当前的运行环境，如果是浏览器，就可以直接操作 `DOM` ，如果是服务器，就需要使用 `Virtual DOM` 生成 `HTML` 字符串。

#### 到了这里仿佛一切都很简单，一切都这么顺其自然，但是问题又出现了，路由怎么办

浏览器路由和服务器路由完全是两种不同的运行机制，`SPA` 浏览器路由机制可以看[这里](https://juejin.im/post/5ba499765188255c6666e619)，其实原因很简单，在服务器端需要通过请求路径，找到路由组件，而在客户端需通过浏览器中的网址，找到路由组件，是完全不同的两套机制，所以这部分代码是肯定无法公用。

所以 `React` 分别为浏览器端和服务器端分别提供了 `BrowserRouter` 和 `StaticRouter` 两种路由，通过 `BrowserRouter` 我们能够匹配到浏览器即将显示的路由组件，对浏览器来说，我们需要把组件转化成 `DOM`，所以需要我们使用 `ReactDom.render` 方法来进行 `DOM` 的挂载。而 `StaticRouter` 能够在服务器端匹配到将要显示的组件，对服务器端来说，我们要把组件转化成字符串，这时我们只需要调用 `ReactDom` 提供的 `renderToString` 方法，就可以得到 `App` 组件对应的 `HTML` 字符串。

#### 那么，现在差不多要完成了吧

还没有！

对于一个 `React` 应用来说，路由一般是整个程序的执行入口。在 `SSR` 中，服务器端的路由和客户端的路由不一样，也就意味着服务器端的入口代码和客户端的入口代码是不同的。而入口则是 `Webpack` 进行打包完成的。

针对代码运行环境的不同，要进行有区别的 `Webpack` 打包，我们需要在 `Webpack` 的配置中加入 `target: 'node'`，表明是服务器环境进行打包，除此之外，还有各种各样的配置需要解决。

#### 等等，万一要用到 `Redux` 来进行状态管理呢

如果要用到 `redux` 进行全局状态管理，一定要记得写成这种形式：

```js
const getStore = req => {
  return createStore(reducer, defaultState);
};
export default getStore;
```

因为服务器端的 `Store` 是所有用户都要用的，但是不能让所有用户共享 `Store` ，所以在服务器端渲染中，`Store` 的创建应该像下面这样，返回一个函数，每个用户访问的时候，这个函数重新执行，为每个用户提供一个独立的 `Store`。

#### 最后还有什么注意点吗

由于服务器不存在挂载元素这一生命周期，所以例如 `React` 的 `componentDidMount` 或者 `VUE` 的 `mounted` 生命周期都不会执行了，所以在服务端利用接口获取数据的时候，不能写入上述的生命周期中。

## 最后总结

1. 如果页面无数据，或者是纯静态页面，建议使用 `Prerender`，这是一种通过预览打包的方式构建页面，也不会增加服务器负担，但其他情况并不推荐。
2. 当访问量过大时，`SSR` 的实时构建会加剧服务器 `CPU` 的消耗，需结合其他技术进行处理（例如 CDN，服务器缓存，负载均衡等）
3. 如果页面数据请求多，又对 `SEO` 和加载速度有需求的，建议使用 `SSR`
4. 对于高操作需求的项目来说，`CSR` 可能更加适合，页面显示元素即绑定了操作，而 `SSR` 和 `Prerender` 虽然会提前显示页面，但此时页面元素无法操作，仍需要下载完 `bundle.js` 进行事件绑定才能执行

> 当然在真正实现 `SSR` 架构的过程中，难点有时不是实现的思路，而是细节的处理。比如说如何针对不同页面设置不同的 `title` 和 `description` 来提升 `SEO` 效果，这时候，我们其实可以用 `react-helmet` 这样的工具帮我们达成目标，这个工具对客户端和服务器端渲染的效果都很棒，值得推荐。还有一些诸如工程目录的设计，404，301 重定向情况的处理等等，不过这些问题，我们只需要在实践中遇到的时候逐个攻破就可以了。
