## [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)


内容安全策略(CSP)是一种附加的安全层，可以帮助检测和减轻某些类型的攻击，包括跨站脚本攻击和数据注入攻击。这些攻击被用于数据盗窃, 站点破坏以及恶意软件的传播。

---

CSP被设计为完全向后兼容, CSP版本2除外，其中有一些明确提到的向后兼容性的不一致性;更多[细节](https://www.w3.org/TR/CSP2/)请参见1.1。不支持CSP的浏览器仍然可以使用实现CSP的服务器，反之亦然:不支持CSP的浏览器只是简单地忽略CSP，像往常一样运行，默认使用web内容的标准同源策略。如果站点不提供CSP头，浏览器同样使用标准的同源策略。


要启用CSP，您需要配置web服务器以返回Content-Security-Policy HTTP头(有时您会看到提到X-Content-Security-Policy头，但这是一个旧版本，您不再需要指定它)。


或者，可以使用meta标签配置策略，例如

```
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src https://*; child-src 'none';">
```

---

### CSP功能

- 减轻跨站脚本

CSP的主要目标是减少和报告XSS攻击。XSS攻击利用浏览器对从服务器接收的内容的信任。恶意脚本是由受害者的浏览器执行的，因为浏览器信任内容的来源，即使它看起来不是来自哪里。


CSP允许服务器管理员通过指定浏览器认为是可执行脚本有效来源的域来减少或消除XSS载体。然后，与CSP兼容的浏览器将只执行从这些白名单域接收的源文件中加载的脚本，而忽略所有其他脚本(包括内联脚本和事件处理HTML属性)。


作为一种最终的保护形式，那些希望永远不允许执行脚本的站点可以选择全局不允许脚本执行。


- 减轻包嗅探攻击

除了限制加载内容的来源域之外，服务器还可以指定允许使用哪些协议;例如(从安全的角度来看，最好是这样)，服务器可以指定所有内容必须使用HTTPS加载。完整的数据传输安全策略不仅包括强制使用HTTPS进行数据传输，而且还使用安全标志标记所有cookie，并提供从HTTP页到HTTPS页的自动重定向。站点还可以使用严格传输安全的HTTP报头，以确保浏览器仅通过加密的通道连接到它们。

---

### 使用CSP

配置内容安全策略涉及向web页面添加内容安全策略HTTP头，传递赋值，使用赋值控制用户代理加载页面需要的被授权的资源。例如，上传和显示图像的页面可以允许来自任何地方的图像，但是将表单操作限制为特定的端点。设计适当的内容安全策略有助于保护页面免受跨站点脚本攻击。本文将解释如何正确地构造这些头文件，并提供示例。


- 指定你的政策

您可以使用内容安全策略HTTP头来指定您的策略，如下所示:


    Content-Security-Policy:策略

策略是包含描述内容安全策略的策略指示的字符串。


- 写一个(内容安全)策略

策略是使用一系列策略指令(policy directives)进行描述，每个策略指令描述特定资源类型或策略区域的策略。您的策略应该包含一个[default-src](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/default-src)策略指令，这是其他资源类型在没有自己的策略时的一种降级(对于完整的列表，请参见default-src指令的描述)。一个策略需要包含一个default-src或script-src指令来防止内联脚本运行，以及阻止eval()的使用。策略需要包含一个default-src或style-src指令，以限制内联样式通过DOM 'script 标签'节点或DOM节点的style属性加载。

---

### 举例： 常规示例

本节提供一些常见安全策略场景的示例。

- 示例1

网站管理员希望所有内容都来自网站本身(不包括子域)。


    Content-Security-Policy: default-src 'self'

- 示例2

web站点管理员希望允许来自受信任域及其所有子域的内容(它不必是CSP所设置的相同域)。

    Content-Security-Policy: default-src 'self' *.trusted.com

- 示例3

web站点管理员希望允许web应用程序的用户在自己的内容中包含来自任何来源的图像，但是要将音频或视频媒体(来源)限定为可靠的提供者，而所有脚本只限定托管可信代码的特定服务器。

    Content-Security-Policy: default-src 'self'; img-src *; 
    media-src media1.com media2.com; script-src userscripts.example.com

在这里，默认情况下，内容仅来自文档源，但有以下例外:


    - 图像可以从任何地方加载(注意“*”通配符)。

    - 媒体只允许来自media1.com和media2.com (不允许来自这些站点的子域)。

    - 可执行脚本只允许来自userscripts.example.com。

- 示例4

在线银行站点的web站点管理员希望确保使用SSL加载其所有内容，以防止攻击者偷听请求.

    Content-Security-Policy: default-src https://onlinebanking.jumbobank.com

服务器只允许访问通过HTTPS通过单一来源的onlinebanking.jumbobank.com加载的文档。

- 示例5

一个web邮件站点的网站管理员希望在电子邮件中允许HTML，以及从任何地方加载的图片，而不是JavaScript或其他潜在的危险内容。

    Content-Security-Policy: default-src 'self' *.mailsite.com; img-src *

注意，这个例子没有指定脚本script-src;通过示例CSP，这个站点使用默认的src指令指定的设置，这意味着脚本只能从原始服务器加载。

---

### 测试你的策略

为了方便部署，CSP可以部署在report-only模式中。策略没有被强制执行，但是任何违反行为都被报告给提供的URI。此外，可以使用仅报告的头来测试将来对策略的修订，而不必实际部署它。


您可以使用Content-Security-Policy-Report-Only HTTP报头来指定您的策略，如下所示:


    Content-Security-Policy-Report-Only:策略...

如果在相同的响应中出现了Content-Security-Policy-Report-Only和 Content-Security-Policy标头，那么这两种策略都是值得尊敬的。在content - security-policy header中指定的策略被执行，而content-security-policy-report-only策略生成报告，但不会被执行。

---

### 启用报告

默认情况下，不会发送违例(violation)报告。要启用违例报告，您需要指定report-URI指令，提供至少一个URI，以便向其交付报告:

    Content-Security-Policy: default-src 'self'; 
    report-uri http://reportcollector.example.com/collector.cgi

然后需要设置服务器来接收报告;它可以以任何你认为合适的方式存储或处理它们。

---

### 违例报告语法

报告JSON对象包含以下数据:

- blocked-uri

    被内容安全策略阻止的资源的URI。如果被阻塞的URI与文档URI属于不同的源，那么被阻塞的URI将被截断，只包含概要、主机和端口。

- disposition

    根据Content-Security-Policy-Report-Only 头或 Content-Security-Policy 头，决定执行“执行”或“报告”。

- document-uri

    出现违例的文档的URI。

- effective-directive

    引发违例执行的指令。

- origin-policy

    内容安全策略HTTP头指定的原始策略。

- referrer

    发生违法行为的文件的引用人。

- script-sample

    引起违规的内联脚本、事件处理程序或样式的前40个字符。

- status-code

    实例化全局对象的资源的HTTP状态码。

- violated-directive

    被违反的策略部分的名称。

---

### 违例报告样本

假如一个位于`http://example.com/signup.html`的页面。它使用以下策略，除了cdn.example.com中的样式表外，不允许其他任何内容。

    Content-Security-Policy: default-src 'none'; 
    style-src cdn.example.com; report-uri /_/csp-reports

html文件 signup.html如下

```
<!DOCTYPE html>
<html>
  <head>
    <title>Sign Up</title>
    <link rel="stylesheet" href="css/style.css">
  </head>
  <body>
    ... Content ...
  </body>
</html>
```

你能找出错误吗?样式表只允许从cdn.example.com加载，但是该网站试图从它自己的源`http://example.com`加载样式表。有能力执行CSP的浏览器会在访问文档时将以下的违规报告发送到`http://example.com/_/csp-reports上`:

```
{
  "csp-report": {
    "document-uri": "http://example.com/signup.html",
    "referrer": "",
    "blocked-uri": "http://example.com/css/style.css",
    "violated-directive": "style-src cdn.example.com",
    "original-policy": "default-src 'none'; style-src cdn.example.com; report-uri /_/csp-reports"
  }
}
```

你看，该报告包含blocked-uri中违反资源的完整路径。情况并非总是如此。例如，当注册时。html将尝试从`http://anothercdn.example.com/stylesheet.css`中加载CSS，该浏览器将不包括完整路径，而只包括起源`http://anothercdn.example.com`。[CSP规范解释了这种奇怪的行为](https://www.w3.org/TR/CSP/#security-violation-reports)。总之，这样做是为了防止关于跨源资源的敏感信息泄露。

---

### 浏览器兼容性


- 客户端

| feature | chrome | edge | firefox | ie | opera | safari| 
| ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| content-security-policy |25|14 |23|10|15|7|
| base-uri |40|NO |35|NO|27| 10 |
| block-all-mixed-content |YES|? |48|NO|YES|?|
| child-src |40| 15 | 45 | NO | 27 | 10 |
| connect-src |25|14 |23|NO|15|7|
| default-src |25|14 |23|NO|15|7|
| disown-opener |NO|NO |NO|NO|NO|NO|
| font-src |25|14 |23|NO|15|7|
| form-action |40|15 |36|NO|27|10|
| frame-ancestors |40|15|33|NO|26|10|
| frame-src |25|14 |23|NO|15|7|
| img-src |25|14 |23|NO|15|7|
| manifest-src |YES|NO|41|NO|YES|NO|
| media-src |25|14|23|NO|15|7|
| navigation-to |NO|NO|NO|NO|NO|NO|
| object-src |25|14 |23|NO|15|7|
| plugin-types |40|15|NO|NO|27|10|
| referrer |!(33-56)|NO|37|NO|!(?-43)|NO|
| report-sample |59|?|?|?|?|?|
| report-to | NO | NO | NO | NO | NO | NO |
| report-uri | 25 | 14 | 23 | NO | 15 | 7 |
| require-sri-for | 54 | NO | 49 | NO | 41 | NO |
| sandbox | 25 | 14 | 50 | 10 | 15 | 7 |
| script-src | 25 | 14 | 23 | NO | 15 | 7 |
| strict-dynamic | 52 | NO | 52 | NO | 39 | NO |
| style-src | 25 | 14 | 23 | NO | 15 | 7 |
| upgrade-insecure-request | 43 | NO | 42 | NO | 30 | NO |
| worker-src | 59 | NO | 58 | NO | 48 | NO |

- 移动端

| feature | android webview | android chrome | edge mobile | android firefox | android opera | ios safari | sumsung internet | 
| ------ | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| content-security-policy |YES|YES|YES|23|?|7.1| YES |
| base-uri |YES|YES|NO|35|?| 9.3 | YES|
| block-all-mixed-content |YES|YES|?|48|?|?|YES|
| child-src |YES| YES | NO | 45 | ? | 9.3 | YES|
| connect-src |YES|YES |?|23|?|7.1|YES| 
| default-src |YES|YES |?|23|?|7.1|YES| 
| disown-opener |NO|NO |NO|NO|NO|NO|NO|
| font-src |YES|YES |?|23|?|7.1|YES| 
| form-action |YES|YES |NO|36|?|9.3|YES| 
| frame-ancestors |?|YES|NO|33|?|9.3|YES|
| frame-src |YES|YES |?|23|?|7.1|YES|
| img-src |YES|YES |?|23|?|7.1|YES|
| manifest-src |YES|YES |NO|41|?|NO|YES|
| media-src |YES|YES |?|23|?|7.1|YES|
| navigation-to |NO|NO|NO|NO|NO|NO|
| object-src |YES|YES |?|23|?|7.1|YES|
| plugin-types |YES|YES |NO|NO|?|9.3|YES|
| referrer |!(33-56)|!(33-56)|NO|37*|!(?-43)|NO|YES|
| report-sample |59|59|?|?|46|?|7.0|
| report-to |NO|NO|NO|NO|NO|NO|NO|
| report-uri |YES|YES |?|23|?|7.1|YES|
| require-sri-for |54|54 |NO|49|41|NO|6.0|
| sandbox |YES|YES |?|50|?|7.1|YES|
| script-src |YES|YES |?|23|?|7.1|YES|
| strict-dynamic |52|52|NO|NO|39|NO|6.0|
| style-src |YES|YES |?|23|?|7.1|YES|
| upgrade-insecure-request |43|43|NO|42|30|NO|4.0|
| worker-src |59*|59*|NO|58|48|NO|7.0|

在一些版本的Safari浏览器中存在不兼容,即如果设置了一个内容安全政策头,但不是一个同源头,浏览器将阻止自托管内容和站外内容,和错误报告,这是由于内容安全策略不允许的内容。


- [详细图表可参考链接](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)

---

### 相关资源

- [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)

- [Content-Security-Policy-Report-Only](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only)

- [Content Security in WebExtensions](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/Content_Security_Policy)

- [Display security and privacy policies In Firefox Developer Tools](https://developer.mozilla.org/en-US/docs/Tools/GCLI/Display_security_and_privacy_policies)
