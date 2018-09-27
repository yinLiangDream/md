---
typora-copy-images-to: ipic
---

跨站脚本攻击（XSS）是客户端脚本安全的头号大敌。本文章深入探讨 XSS 攻击原理，下一章（XSS 攻击进阶）将深入讨论 XSS 进阶攻击方式。

本系列将持续更新。

## XSS 简介

XSS（Cross Site Script），全称跨站脚本攻击，为了与 CSS（Cascading Style Sheet） 有所区别，所以在安全领域称为 XSS。

XSS 攻击，通常指黑客通过 **HTML 注入** 篡改网页，插入恶意脚本，从而在用户浏览网页时，控制用户浏览器的一种攻击行为。在这种行为最初出现之时，所有的演示案例全是跨域行为，所以叫做 "**跨站脚本**" 。时至今日，随着 Web 端功能的复杂化，应用化，是否跨站已经不重要了，但 XSS 这个名字却一直保留下来。

随着 Web 发展迅速发展，JavaScript 通吃前后端，甚至还可以开发 APP，所以在产生的应用场景越来越多，越来越复杂的情况下， XSS 愈来愈难统一针对，现在业内达成的共识就是，针对不同的场景而产生的不同 XSS ，需要区分对待。可即便如此，复杂应用仍然是 XSS 滋生的温床，尤其是很多企业实行迅捷开发，一周一版本，两周一大版本的情况下，忽略了安全这一重要属性，一旦遭到攻击，后果将不堪设想。

那什么是 XSS 呢？我们看下面一个例子。

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>XSS</title>
</head>

<body>
  <div id="t"></div>
  <input id="s" type="button" value="获取数据" onclick="test()">
</body>
<script>
  function test() {
    // 假设从后台取出的数据如下
    const arr = ['1', '2', '3', '<img src="11" onerror="alert(\'我被攻击了\')" />']
    const t = document.querySelector('#t')
    arr.forEach(item => {
      const p = document.createElement('p')
      p.innerHTML = item
      t.append(p)
    })
  }
</script>

</html>
```

这个时候我们在页面上点击 `获取数据` 按钮时，页面上会出现如下信息：

![image-20180911193528855](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-09-26-030516.png)

你会发现，本应该作为数据展示在界面上的内容居然执行了，这显然是开发者不希望看到的。

## XSS 攻击类型

XSS 根据效果的不同可以分为如下几类：

### 反射型 XSS

简单来说，反射型 XSS 只是将用户输入的数据展现到浏览器上（从哪里来到哪里去），即需要一个发起人（用户）来触发黑客布下的一个陷阱（例如一个链接，一个按钮等），才能攻击成功，一般容易出现在搜索页面、留言板块。这种反射型 XSS 也叫做 **非持久型 XSS（No-persistent XSS）** 。

例如：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>

<body>
  <div id="t"></div>
  <input id="s" type="button" value="获取数据" onclick="test()">
</body>
<script>
  function test() {
    const arr = ['1', '2', '3', '<img src="11" onerror="console.log(window.localStorage)" />']
    const t = document.querySelector('#t')
    arr.forEach(item => {
      const p = document.createElement('p')
      p.innerHTML = item
      t.append(p)
    })
  }
</script>

</html>
```

假设这是一个留言板块，加载到这一页时，页面会输出：

![image-20180911200903074](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%80%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E5%9F%BA%E7%A1%80%E5%8F%8A%E5%8E%9F%E7%90%86.assets/image-20180911200903074.png?raw=true)

黑客可以轻易盗取存储在你本地浏览器的各种信息，进而模拟登陆信息，黑入账户，进行各种操作。

### 存储型 XSS

存储型 XSS 会把用户输入的数据 _保存_ 在服务器端，这种 XSS 十分稳定，十分有效，效果持久。存储型 XSS 通常叫做 "**持久型 XSS（Persistent XSS）**"，即存在时间比较长。

比较常见的场景就是，黑客写下一篇包含恶意代码的文章，文章发表后，所有访问该博客文章的用户都会执行这一段代码，进行恶意攻击。

例如：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>

<body>
  <div id="t">
    这是我写的一篇文章
  </div>
</body>
<script>
  console.log(navigator.userAgent)
</script>

</html>
```

![image-20180911202010076](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%80%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E5%9F%BA%E7%A1%80%E5%8F%8A%E5%8E%9F%E7%90%86.assets/image-20180911202010076.png?raw=true)

直接输出了浏览器信息，黑客可以获取到这些信息后，发送到自己的服务器，随意操作。

### DOM Based XSS

实际上，这种类型的 XSS 与是否存储在服务器端无关，从效果上来说也是反射型 XSS，单独划分出来是因为此类 XSS 形成的原因比较特殊。

简单来说，通过修改页面 DOM 节点形成的 XSS，称之为 DOM Based XSS。

例子如下：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>XSS</title>
</head>

<body>
  <div id="t"></div>
  <input type="text" id="text" value="">
  <input type="button" id="s" value="search" onclick="test()">
</body>
<script>
function test() {
  const str = document.querySelector('#text').value
  document.querySelector('#t').innerHTML = '<a href="' + str + '" >查找结果</a>'
}
</script>

</html>
```

该页面的作用是，在输入框内输入一个内容，跳出查找结果能直接跳转，效果如下：

![image-20180911204357177](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%80%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E5%9F%BA%E7%A1%80%E5%8F%8A%E5%8E%9F%E7%90%86.assets/image-20180911204357177.png?raw=true)

点击查找结果后，页面会自动跳到百度（毒）页面，但是细心的我们会发现，这字符串拼接有可乘之机啊，输入`" onclick=alert(/XSS/) //`：

![image-20180911204628206](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%80%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E5%9F%BA%E7%A1%80%E5%8F%8A%E5%8E%9F%E7%90%86.assets/image-20180911204628206.png?raw=true)

果然，页面执行了我们输入的东西，上面的内容是，第一个双引号闭合掉`href`的第一个双引号，然后插入`onclick`事件，最后注释符 `//`注释掉第二个双引号，点击跳转链接，脚本就被执行了。
