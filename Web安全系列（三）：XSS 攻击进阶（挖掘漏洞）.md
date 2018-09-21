# 前言

在前些章节 [（web 安全系列（一）：XSS 攻击基础及原理）](https://yq.aliyun.com/articles/638829?spm=a2c4e.11155435.0.0.59893312yiELAF)以及[（Web 安全系列（二）：XSS 攻击进阶（初探 XSS Payload））](https://yq.aliyun.com/articles/638935?spm=a2c4e.11155435.0.0.6d253312lHXJ4V)中，我详细介绍了 XSS 形成的原理以及 XSS 攻击的分类，并且编写了一个小栗子来展示出 `XSS Payload` 的危害。

目前来说，XSS 的漏洞类型主要分为三类：反射型、存储型、DOM 型，在本篇文章当中会以`permeate`生态测试系统为例，分析网站功能，引导攻击思路，帮助读者能够快速找出网站可能存在的漏洞。

# 反射型 XSS 挖掘

现在笔者需要进行手工 XSS 漏洞挖掘，在手工挖掘之前笔者需要先逛逛网站有哪些功能点，如下图是`permeate`的界面

![下载](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/%E4%B8%8B%E8%BD%BD.png?raw=true)

## 思路分析

我们知道反射型 XSS ，大多是通过 URL 传播的，那么我就需要思考哪些地方会出现让 URL 地址的参数在页面中显示，我相信大部分读者脑中第一直觉就是搜索栏，尤其是一些大型网站的站内搜索，搜索的关键词会展示在当前的页面中。例如某搜索引擎：

![image-20180917170314833](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917170314833.png?raw=true)

而在我们测试的首页也有网站搜索功能，因此我们可以从搜索功能着手测试，尝试是否可以进行 XSS Payload，我们先输入一个简单的 Payload 进行测试，测试代码为`<img onerror="alert(1)" src=1 />`

当我们点击搜索按钮时，URL 应当自动改变为 `http://localhost:8888/home/search.php?keywords=<img onerror="alert(1)" src=1 />`

## 操作过程

我们进行尝试：

先输入搜索内容

![image-20180917170907957](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917170907957.png?raw=true)

再进行搜索

![image-20180917171035872](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917171035872.png?raw=true)

我们发现，Google Chrome 居然直接阻止了该事件，Payload 也没有进行触发，这里我就需要跟读者说一下了，Chrome 浏览器的内核自带 XSS 筛选器，对于反射型 XSS 会自动进行拦截，所以尽量不要用 Chrome 进行测试，我们改用火狐继续进行测试：

![image-20180917171414387](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917171414387.png?raw=true)

果然，直接触发了我们的 Payload 。

## 结果分析

此 Payload 被触发，说明我们找到了一个反射型 XSS 的漏洞，当然，这种漏洞非常初级，绝大部分网站都进行了过滤操作，再加上随着浏览器功能越来越强大，浏览器自带的 XSS 筛选器变得更加智能，这种漏洞会越来越少见，下面我将会测试更为常见的存储型 XSS 的挖掘与并介绍如何绕过。

# 存储型 XSS 挖掘

## 思路分析

存储型 XSS 的攻击代码是存在服务器端，因此，我们需要找到该网站将数据存储到后端的功能，我们对此网站有了一定了解，会发现 `permeate` 拥有发帖和回帖功能，这正是 Web 端和后台进行沟通的渠道，所有帖子信息都会存在服务端，有了这些信息，我们可以进入板块，进行发帖操作：

![image-20180917172547131](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917172547131.png?raw=true)

进入发帖界面：

![image-20180917172646510](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917172646510.png?raw=true)

## 检测漏洞

我们现在标题和内容里填上初级的 Payload ：`123<script>alert('123')</script>`

![image-20180917202811060](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917202811060.png?raw=true)

我们进行发表操作：

![image-20180917202825918](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917202825918.png?raw=true)

页面直接执行了我们的 Payload，我们点完确定，查看列表：

![image-20180917202841327](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917202841327.png?raw=true)

我们进入帖子内部，会发现如下场景：

![image-20180917203220506](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917203220506.png?raw=true)

很明显，文章主体部分的 Payload 并没有执行，这到底是怎么一回事呢？

## 抓包

为什么标题内容可以 Payload，主体内容不能 Payload 呢，我们打开控制台，切到`Network` 再来发一篇帖子看看：

![image-20180917203838605](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917203838605.png?raw=true)

我们可以看到对应的内容已经经过转义，转义分为两种，前端转义和后端转义，如果是后端转义通常我们就不需要测试下去了，因为我们不知道服务端的内部代码，如果是前端转义则可以绕过这个限制。

那么该如何操作呢？

![image-20180917204208148](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917204208148.png?raw=true)

我们拷贝出 URL

`curl 'http://localhost:8888/home/_fatie.php?bk=5&zt=0' -H 'Connection: keep-alive' -H 'Cache-Control: max-age=0' -H 'Origin: http://localhost:8888' -H 'Upgrade-Insecure-Requests: 1' -H 'Content-Type: application/x-www-form-urlencoded' -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.92 Safari/537.36' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8' -H 'Referer: http://localhost:8888/home/fatie.php?bk=5' -H 'Accept-Encoding: gzip, deflate, br' -H 'Accept-Language: zh-CN,zh;q=0.9,en;q=0.8' -H 'Cookie: PHPSESSID=7690435026da386df8a80e63f3da2089' --data 'csrf_token=9191&bk=5&title=123%3Cscript%3Econsole.log%28232%29%3C%2Fscript%3E&content=%3Cp%3E123%26lt%3Bscript%26gt%3Bconsole.log%28232%29%26lt%3B%2Fscript%26gt%3B%3C%2Fp%3E' --compressed`

找到其中的 title 和 content，将 content 的内容替换为 title 的内容：

`curl 'http://localhost:8888/home/_fatie.php?bk=5&zt=0' -H 'Connection: keep-alive' -H 'Cache-Control: max-age=0' -H 'Origin: http://localhost:8888' -H 'Upgrade-Insecure-Requests: 1' -H 'Content-Type: application/x-www-form-urlencoded' -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.92 Safari/537.36' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8' -H 'Referer: http://localhost:8888/home/fatie.php?bk=5' -H 'Accept-Encoding: gzip, deflate, br' -H 'Accept-Language: zh-CN,zh;q=0.9,en;q=0.8' -H 'Cookie: PHPSESSID=7690435026da386df8a80e63f3da2089' --data 'csrf_token=9191&bk=5&title=123%3Cscript%3Econsole.log%28232%29%3C%2Fscript%3E&content=123%3Cscript%3Econsole.log%28232%29%3C%2Fscript%3E' --compressed`

替换完成之后，将此内容复制到终端进行运行：

![image-20180917205741958](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917205741958.png?raw=true)

回到主页面查看相关内容：

![image-20180917205958914](https://github.com/yinLiangDream/md/blob/master/Web%E5%AE%89%E5%85%A8%E7%B3%BB%E5%88%97%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AXSS%20%E6%94%BB%E5%87%BB%E8%BF%9B%E9%98%B6%EF%BC%88%E6%8C%96%E6%8E%98%E6%BC%8F%E6%B4%9E%EF%BC%89.assets/image-20180917205958914.png?raw=true)

Payload 执行了两次，内容也被攻击了。

## 结果分析

看到此处说明我们已经成功绕过前端 XSS 过滤器，直接对内容进行修改，所以后端的转义有时候也很有必要。

# 总结

挖掘漏洞是一个复杂的过程，手工挖掘不失为一种可靠的方式，但是手动挖掘效率低下，有时还要看运气，目前已经出现很多自动检测 XSS 漏洞的工具以及平台，大大提高发现漏洞的效率，我将在稍后的章节中介绍一些工具以及如何防御 XSS。
