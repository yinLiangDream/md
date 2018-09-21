# 什么是 XSS Payload

上一章我谈到了 XSS 攻击的几种分类以及形成的攻击的原理，并举了一些浅显的例子，接下来，我就阐述什么叫做 `XSS Payload` 以及从攻击者的角度来初探 XSS 攻击的威力。

在黑客 XSS 攻击成功之后，攻击者能够对用户当前浏览的页面植入各种恶意脚本，通过恶意脚本来控制浏览器，这些脚本实质上就是 `JavaScript` 脚本（或者是其他浏览器可以运行的脚本），这种恶意植入且具有完成各种具体功能的恶意脚本就被称为 **`XSS Payload`**。

# 初探 XSS Payload

一个最常见的 `XSS Payload` ，就是通过浏览器读取 Cookie 对象，进而发起 `Cookie 劫持` 攻击。

一般一个网站为了防止用户无意间关闭页面，重新打开需要重新输入账号密码繁杂的情况下，一般都会把登录信息（登录凭证）加密存储在 CooKie 中，并且设置一个超时时间，在此时间段内，用户利用自己账号信息随意进出该网站。如果该网站遭到 `XSS Payload` ，黑客盗取了该用户的 Cookie 信息，往往意味着该用户的登录凭证丢失了，换句话说，攻击者不需要知道该用户的账号密码，直接利用盗取的 Cookie 模拟凭证，直接登录到该用户的账户。

如下所示，攻击者先在一个社区发表一篇文章：

![image-20180912170017834](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E5%88%9D%E6%8E%A2%20XSS%20Payload%EF%BC%89.assets/image-20180912170017834.png?raw=true)

你有意无意点了一下 `点我得大奖` 这个时候，`XSS Payload` 就生效了：

![image-20180912170617607](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E5%88%9D%E6%8E%A2%20XSS%20Payload%EF%BC%89.assets/image-20180912170617607.png?raw=true)

![image-20180912170131327](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E5%88%9D%E6%8E%A2%20XSS%20Payload%EF%BC%89.assets/image-20180912170131327.png?raw=true)

该`XSS Payload`会请求一个 img 图片，图片请求地址即为黑客的服务器地址， url 参数带上 Cookie ，我们在后台服务器接收到了这个请求：

![image-20180912170440048](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E5%88%9D%E6%8E%A2%20XSS%20Payload%EF%BC%89.assets/image-20180912170440048.png?raw=true)

这个时候，黑客就可以获取到此 Cookie，然后模拟 CooKie 登陆。

当然传输的内容可以是任何内容，只要能获取到的，全都可以传输给后台服务器。

如何利用窃取的 Cookie 登陆目标用户的账户呢？这和`利用自定义Cookie访问网站`的过程是一样的，参考如下：

当没有登陆的时候，Cookie 内容是空的：

![image-20180912171431847](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E5%88%9D%E6%8E%A2%20XSS%20Payload%EF%BC%89.assets/image-20180912171431847.png?raw=true)

当我们手动添加 Cookie 后，登陆的内容如下：

![image-20180912171600286](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E5%88%9D%E6%8E%A2%20XSS%20Payload%EF%BC%89.assets/image-20180912171600286.png?raw=true)

此时，我们就已经登陆上了该用户的账户。

所以，通过 XSS 攻击，可以完成 `Cookie 劫持` 攻击，直接登陆进用户的账户。

其实都不需要带上参数，黑客就能获取到所有数据，这是因为当前 Web 中，Cookie 一般是用户凭证，浏览器发起的所有请求都会自动带上 Cookie 。

![image-20180912172448268](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E5%88%9D%E6%8E%A2%20XSS%20Payload%EF%BC%89.assets/image-20180912172448268.png?raw=true)

![image-20180912172459326](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%BA%8C%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E5%88%9D%E6%8E%A2%20XSS%20Payload%EF%BC%89.assets/image-20180912172459326.png?raw=true)

那么该如何预防 `Cookie 劫持` 呢？

Cookie 的 `HttpOnly` 标识可以有效防止 `Cookie 劫持`，我们会在稍后章节具体介绍。
