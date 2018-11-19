# Web 安全漏洞之 XSS 攻击

## 什么是 XSS 攻击

XSS（Cross-Site Scripting）又称跨站脚本，XSS的重点不在于跨站点，而是在于脚本的执行。XSS是一种经常出现在 Web 应用程序中的计算机安全漏洞，是由于 Web 应用程序对用户的输入过滤不足而产生的。

## 常见的三种 XSS 攻击类型

常见的 XSS 攻击有三种：反射型、DOM-based 型、存储型。 其中反射型、DOM-based 型可以归类为非持久型 XSS 攻击，存储型归类为持久型 XSS 攻击。

### 1.反射型

反射型 XSS 一般是攻击者通过特定手法（如电子邮件），诱使用户去访问一个包含恶意代码的 URL，当受害者点击这些专门设计的链接的时候，恶意代码会直接在受害者主机上的浏览器执行。

对于访问者而言是一次性的，具体表现在我们把我们的恶意脚本通过 URL 的方式传递给了服务器，而服务器则只是不加处理的把脚本“反射”回访问者的浏览器而使访问者的浏览器执行相应的脚本。反射型 XSS 的触发有后端的参与，要避免反射性 XSS，必须需要后端的协调，后端解析前端的数据时首先做相关的字串检测和转义处理。

此类 XSS 通常出现在网站的搜索栏、用户登录口等地方，常用来窃取客户端 Cookies 或进行钓鱼欺骗。

整个攻击过程大约如下：

![反射型][https://p4.ssl.qhimg.com/t01a98b7f389ece8c01.png]

### 2.DOM-based 型

客户端的脚本程序可以动态地检查和修改页面内容，而不依赖于服务器端的数据。例如客户端如从 URL 中提取数据并在本地执行，如果用户在客户端输入的数据包含了恶意的 JavaScript 脚本，而这些脚本没有经过适当的过滤和消毒，那么应用程序就可能受到 DOM-based XSS 攻击。需要特别注意以下的用户输入源 document.URL、location.hash、location.search、document.referrer 等。

整个攻击过程大约如下：

![DOM-based][https://p4.ssl.qhimg.com/t0165dddb475fc1e83c.png]

### 3.存储型

攻击者事先将恶意代码上传或储存到漏洞服务器中，只要受害者浏览包含此恶意代码的页面就会执行恶意代码。这就意味着只要访问了这个页面的访客，都有可能会执行这段恶意脚本，因此储存型XSS的危害会更大。

存储型 XSS 一般出现在网站留言、评论、博客日志等交互处，恶意脚本存储到客户端或者服务端的数据库中。

整个攻击过程大约如下：

![DOM-based][https://p0.ssl.qhimg.com/t017b61091f1c67ca05.png]

## XSS 的危害

 XSS 可以导致攻击劫持访问；盗用 cookie 实现无密码登录；配合 csrf 攻击完成恶意请求；使用 js 或 css 破坏页面正常的结构与样式等。

## XSS 的防护

1. XSS 防御之 HTML 编码

应用范围：将不可信数据放入到 HTML 标签内（例如div、span等）的时候进行HTML编码。

编码规则：将 & < > " ' / 转义为实体字符（或者十进制、十六进制）。

示例代码：

```js
  function encodeForHTML(str, kwargs){
    return ('' + str)
      .replace(/&/g, '&amp;')
      .replace(/</g, '&lt;')     // DEC=> &#60; HEX=> &#x3c; Entity=> &lt;
      .replace(/>/g, '&gt;')
      .replace(/"/g, '&quot;')
      .replace(/'/g, '&#x27;')   // &apos; 不推荐，因为它不在HTML规范中
      .replace(/\//g, '&#x2F;');
  };
```

HTML 有三种编码表现方式：十进制、十六进制、命名实体。例如小于号（<）可以编码为 "十进制> &#60;", "十六进制=> &#x3c;", "命名实体=> &lt;" 三种方式。对于单引号（'）由于实体字符编码方式不在 HTML 规范中，所以此处使用了十六进制编码。


2. XSS 防御之 HTML Attribute 编码

应用范围：将不可信数据放入 HTML 属性时（不含src、href、style 和事件处理属性），进行 HTML Attribute 编码

编码规则：除了字母数字字符以外，使用 &#xHH;(或者可用的命名实体)格式来转义ASCII值小于256所有的字符​​​​​​​

示例代码：

```js
  function encodeForHTMLAttibute(str, kwargs){
    let encoded = '';
    for(let i = 0; i < str.length; i++) {
      let ch = hex = str[i];
      if (!/[A-Za-z0-9]/.test(str[i]) && str.charCodeAt(i) < 256) {
        hex = '&#x' + ch.charCodeAt(0).toString(16) + ';';
      }
      encoded += hex;
    }
    return encoded;
  };
```

3. XSS 防御之 JavaScript 编码

作用范围：将不可信数据放入事件处理属性、JavaScirpt值时进行 JavaScript 编码

编码规则：除字母数字字符外，请使用\xHH格式转义ASCII码小于256的所有字符

示例代码：

```js
  function encodeForJavascript(str, kwargs) {
    let encoded = '';
    for(let i = 0; i < str.length; i++) {
      let cc = hex = str[i];
      if (!/[A-Za-z0-9]/.test(str[i]) && str.charCodeAt(i) < 256) {
        hex = '\\x' + cc.charCodeAt().toString(16);
      }
      encoded += hex;
    }
    return encoded;
  };
```

4. XSS 防御之 URL 编码

作用范围：将不可信数据作为 URL 参数值时需要对参数进行 URL 编码

编码规则：将参数值进行 encodeURIComponent 编码

示例代码：

```js
  function encodeForURL(str, kwargs){
    return encodeURIComponent(str);
  };
```
5. XSS 防御之 CSS 编码

作用范围：将不可信数据作为 CSS 时进行 CSS 编码

编码规则：除了字母数字字符以外，使用\XXXXXX格式来转义ASCII值小于256的所有字符

示例代码：

```js
  function encodeForCSS (attr, str, kwargs){
    let encoded = '';
    for (let i = 0; i < str.length; i++) {
      let ch = str.charAt(i);
      if (!ch.match(/[a-zA-Z0-9]/) {
        let hex = str.charCodeAt(i).toString(16);
        let pad = '000000'.substr((hex.length));
        encoded += '\\' + pad + hex;
      } else {
        encoded += ch;
      }
    }
    return encoded;
  };
```

## 后记

在任何时候用户的输入都是不可信的。对于 HTTP 参数，理论上都要进行验证，例如某个字段是枚举类型，其就不应该出现枚举以为的值；对于不可信数据的输出要进行相应的编码；此外httpOnly、CSP、X-XSS-Protection、Secure Cookie 等也可以起到有效的防护。

XSS 漏洞有时比较难发现，所幸当下React、Vue等框架都从框架层面引入了 XSS 防御机制，一定程度上解放了我们的双手。
但是作为开发人员依然要了解 XSS 基本知识、于细节处避免制造 XSS 漏洞。框架是辅助，我们仍需以人为本，规范开发习惯，提高 Web 前端安全意识。

## 参考文档

[http://www.qa-knowhow.com/?p=1467](http://www.qa-knowhow.com/?p=1467)
[https://brajeshwar.github.io/entities/](https://brajeshwar.github.io/entities/)
[https://excess-xss.com/](https://excess-xss.com/)
[https://github.com/chrisisbeef/jquery-encoder](https://github.com/chrisisbeef/jquery-encoder)