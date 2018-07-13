## [CSP 已死， CSP永生， 论白名单不安全及CSP的未来-译](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/45542.pdf)

    CSP Is Dead, Long Live CSP! On the Insecurity of Whitelists and the Future of Content Security Policy
    - Lukas Weichselbaum  Google Inc. lwe@google.com
    - Michele Spagnuolo Google Inc. mikispag@google.com
    - Artur Janc Google Inc. aaj@google.com
    - Sebastian Lekies Google Inc. slekies@google.com

## 摘要

 
CSP(内容安全策略)一种被设计用于减轻现代web应用头号安全漏洞跨站点脚本(XSS)的web平台安全机制[24]。 在这篇论文中,我们仔细地查看采用CSP的好处和识别现实部署中的重大缺陷， 这些缺陷绕过94.72%的(安全)策略。

我们的全网分析基于搜索引擎资料库中大约来自1百万台主机的10亿张网页，结果覆盖了部署了CSP的1680867台主机名-- 迄今为止最全面的研究。 我们引入了CSP安全相关的各个方面的规范， 提供了CSP隐患模型的深入分析， 聚焦XSS防护， 我们识别三类常见的CSP绕过并说明他们如何破坏策略的安全性。

为了理解CSP的好处， 我们采取定量分析internet上部署的CSP策略， 我们发现15个最常用的加载脚本的白名单域中， 有14个存在不安全的端点， 结果75.81%的策略因使用了白名单被绕过。 总计94.68%的策略使用限制脚本执行测策略无效， 99.34%域采用CSP策略对抗XSS无效。

最后， 我们提议'strict-dynamic'关键字作为补充， 它有助于创建基于随机数加密策略， 不依赖域白名单， 我们讨论了在复杂应用中部署随即加密策略测经验， 为改善策略的web创始人提供指引。

## 关键字

    Content Security Policy; Cross-Site Scripting; Web Security;


## 1. 简介

跨站脚本- 能够向web应用注入攻击者控制的脚本 - 是最为声名狼藉的web漏洞， 自2000年CERT advisory中正式提及， 一代又一代研究者和实践者探索监测， 阻止， 削弱它的方法， 然而， XSS仍然是最普遍的安全问题并随着网络的发展不断变化。

如今，CSP[31]是针对XSS最有希望的对策之一。CSP是一种声明式策略机制，允许web应用程序开发人员定义哪些客户端资源可以由浏览器加载和执行。CSP不允许内联脚本，只允许可信域作为外部脚本的源，它旨在限制站点执行恶意客户端代码的能力。因此，即使攻击者能够发现XSS漏洞，CSP也会通过防止漏洞的利用来保证应用程序的安全性——在无受控制受信任的主机的情况下，攻击者不能够加载恶意代码

在本文中，我们介绍了全网第一次深入分析跨web CSP部署安全性的结果。为了做到这一点，我们首先通过审查CSP的威胁模型、分析可能的配置缺陷和列举不为人知的允许攻击者绕过其保护的技术来研究CSP的保护能力

我们接着使用从谷歌搜索索引中提取的真实世界的 CSP策略进行大规模的实证研究。基于这个数据集，我们发现目前至少有1680,000台Internet主机部署CSP策略。在对我们的数据集进行规范化和去重后，我们识别出26011个不同的CSP策略，其中94.72%是可通过的，攻击者可以使用自动方法找到允许对CSP保护进行破坏的端点。尽管在许多情况下，部署CSP花费了相当大的精力，但当前策略的90.63%的包含允许内联脚本或加载任意外部站点脚本的配置， 这些配置让CSP形同虚设。只有9.37%的策略具有更严格的配置，并且可以潜在地保护抵抗XSS。不幸的是，我们发现这其中至少有51.05%的策略仍然是可以绕过的，这是由于`script-src`白名单中存在细微的策略错误配置或不安全端点源。

基于我们的研究结果，我们认为为一个复杂的应用程序维护一个安全的白名单在实践中是不可行的;因此，我们建议通过指定URL whitelists来指定信任的模型，应该用基于随机数和hashes[3]的方法替换脚本可以执行的方法[0]，该方法已经由CSP规范定义，并且可以在主要的浏览器实现中使用。

在基于随机数的策略中，应用程序定义了一个单用途的、不可猜测的令牌(nonce)，它同时在CSP策略中传递，并且作为合法的、应用程序控制脚本的HTML属性。用户代理只允许执行那些nonce匹配策略中指定值的脚本;能够将标记注入脆弱页面的攻击者不知道nonce值，因此无法执行恶意脚本。为了简化这种基于非基础的方法的采用过程，我们为`script-src`提供了一个新的CSP源表达式，暂时称为`strict-dynamic`。使用`strict-dynamic`，动态生成的脚本会隐式地从创建它们的可信脚本中继承nonce。通过这种方式，已经在执行的合法脚本可以轻松地向DOM添加新脚本，而无需进行大量应用程序更改。但是，发现XSS错误而不知道正确的nonce的攻击者不能滥用这个功能，因为它们首先被阻止执行脚本。

为了证明这种方法的可行性，我们提出了一个在流行的应用程序中采用基于随机数策略的真实案例研究。

我们的贡献可以总结为以下几点：

- 我们介绍了CSP安全模型第一次深入分析的结果，分析了标准对web bug的保护。我们
发现了常见的策略错误配置并呈现三种CSP绕过保护功能的策略。

- 我们通过从谷歌搜索索引中提取策略，对实际CSP部署的好处进行了大规模的实证研究。基于大约1060亿页的语料库，其中39亿页受到CSP的保护，我们确定了26011个独特的策略。我们发现这些政策中至少有94.72%在缓解XSS方面因为配置错误和不安全白名单实体而无效。

- 基于我们的发现，我们建议改变内容安全策略在实践中的部署方式:我们提倡一种基于随机数的方法，而不是使用白名单。为了进一步实现这种方法，我们提出了`strict-dynamic`，这是当前在Chromium浏览器中实现的CSP3规范的新特性。我们讨论了这种方法的好处，并给出了在流行的web应用程序中部署基于随机数和`strict-dynamic`的策略的案例研究。

本文其余部分的结构如下:在第2节中，我们对CSP进行了深入的介绍。因此，我们在2.1中介绍了技术基础、CSP威胁模型和在2.2和2.3中设计策略时常见的安全隐患。随后，我们在第3节中给出了实证研究的结果。为此，我们首先在3.1中概述我们的研究问题，在3.2中介绍我们的数据集，在3.4中介绍结果和分析之前，在3.3中解释我们的方法。基于本研究的结果，我们在第4节提出了一种改进CSP的方法。最后，我们在第5节介绍相关的工作，然后在第6节结束。

## 2. 内容安全策略(CSP)

### 2.1 概述

内容安全策略(Content Security Policy, CSP)是一种声明机制，允许web作者在其应用程序上指定一些安全限制，通过支持用户代理执行策略。

CSP被用作“一种工具，开发人员可以使用它以各种方式锁定他们的应用程序，从而降低内容注入漏洞(…)的风险。”并减少应用程序执行的特权。”[3]

CSP正快速发展:目前最新规范的版本是CSP3，并且该标准由用户代理非均衡实现。例如，Chromium完全支持CSP2，并实现了CSP3的大部分草案，其中有一些(CSP3规范)，还处在实验性的运行时阶段，而Mozilla Firefox和基于webkit的浏览器最近刚刚获得了完全的CSP2支持[8]。在讨论CSP的细节时，我们不关注标准的任何特定修订，而是尝试广泛概述(规范的)应用和规范版本[31]。

CSP策略在`Content-Security-Policy`HTTP响应头或`<meta>`标签中使用。CSP的功能可分为三类:

- 资源加载限制

  CSP最著名和最常用的方面是将各种子资源加载限制到开发人员允许的一组源(即源列表)的能力。常用的指令有`script--src`、`style-src`、`img-src`和接管所有的`default-src`;表1列出了管理资源的指令的完整列表。有一种特殊情况，`script-src`和`style-src`指令有几个附加的配置选项;它们允许对脚本和样式表进行更细粒度的控制，后面将对此进行讨论。

- 基于url的辅助限制

  通过监控获取的子资源不能阻止某些类型的攻击，但同样需要一个可信源概念，可以与之交互文档。一个常见的例子是`frame-ancestors`指令， 为防止clickjacking[10](攻击), 它定义了加载文档的许可源。类似地，`base-uri`和`form-action`定义了哪些url可以作为`<target #href> 以及 <form #action>`的目标，以防止某些`post-xss`攻击[38]

- 各种限制和硬化选项

  由于web应用程序中缺乏其他通用机制启用安全限制，CSP已经成为几个松散安全特性的安寄所。这包括`block-all-mixed-content`和`upgrade-insecure-requests` 关键字，它可以防止混合内容bug和改进HTTPS支持;`plugin-types`，限制允许的插件格式;`sandbox`，它反映了HTML5沙箱框架的安全特性。

`表一：`

|Directive|Controlled Resource type|
|---|---|
|default-src|所有资源(降级)|
|script-src|脚本|
|style-src|样式表|
|img-src|图片|
|media-src|媒体资源(audio, video)|
|font-src|字体|
|frame-src|文档(frames)|
|object-src|插件格式(object, embed)|
|child-src|文档(frames), [shared]workers|
|worder-src|[shared]workers|
|manifext-src|Manifest|

为了使web应用程序与针对XSS的内容安全策略兼容，web作者常常需要重构由应用程序逻辑以及框架和模板系统生成的HTML标记。特别是内联脚本、eval和等效结构的使用、内联事件处理程序和javascript: 必须避免uri或使用对CSP友好替代方法进行重构。

除了执行策略限制的默认行为之外，还可以在`report-only`模式中配置CSP，在这种模式中，攻击行为会被记录下来，但不会被强制执行。在这两种情况下，`report-uri`指令可以用来发送攻击报告，通知应用程序的所有者不兼容的标记。

#### 2.1.1 资源表(source lists)

`listing 1: 传统CSP策略样本`

```
Content-Security-Policy: script-src ’self’; style-src
cdn.example.org third-party.org; child-src https:
```

CSP源列表(通常称为白名单)一直是CSP的核心部分，是传统指定信任关系的方式。例如，如清单1所示，应用程序可能只信任其承载域来加载脚本，但允许使用来自`cdn.example.org`和`third party.org`的字体或图像，并要求通过HTTPS加载框架，同时不限制其他资源类型。

对于任何指令，白名单可以由主机名(`example.org`, `example.com`)组成，可能包括*通配符，扩展对所有子域的信任(`*.example.org`);方案(`https:data:`);以及表示当前文档来源的特殊关键字`self`和`none`，执行空源列表并禁止加载任何资源.

从CSP2开始，作者还可以在他们的白名单中指定路径(`example.org/resources/js/`)。有趣的是，不能依赖基于路径的限制来限制资源的加载位置;关于这个问题的更广泛的讨论载于第2.3.4节。

#### 2.1.2 脚本执行限制

由于脚本在现代web应用程序中的重要性，`script-src`指令提供了几个关键字，以允许对脚本执行进行更细粒度的控制:

|Directive|Controlled Resource type|
|---|---|
|unsafe-inline|允许内联`script`脚本块和javascript事件句柄(可以取消CSP对XSS的保护)执行|
|unsafe-eval|允许使用javascript API, 把数据当成代码执行， 比如`eval()`， `setTimeout()`， `setInterval()` 和 `Function` 构造函数， 这些API被`script-src`指令阻塞|
|CSP nonce|允许策略指定作为脚本授权令牌的一次性值(`script-src`, `nonce-random-value`)。页面上任何带有正确`nonce=“random-value”`属性的脚本都可以执行。|
|CSP hash|允许开发人员在页面中列出预期脚本的密码散列(`script-src`, `sha256-nGA…`)。任何内联脚本，其摘要与策略中提供的值相匹配，将被允许执行。|

类似地，随机数(Nonce)和散列可以与`style-src`指令一起使用，允许通过随机值加载内联样式表和白名单内外部CSS。

`Listing 2: 使用随机数和hash值锁定策略`

```
Content-Security-Policy: script-src ’nonce-BPNLMA4’
’sha256-OPc+f+ieuYDM...’ object-src ’none’;
```
### 2.2 CSP威胁模型

CSP要想提供安全防护，它必须防止攻击者利用漏洞， 对应用程序的用户进行恶意操作。就目前来看，CSP对三种类型漏洞提供防护。

- XSS

  在有漏洞应用程序中注入和执行不受信任的脚本(受`script-src`和`object-src`指令保护)

- Clickjacking

  通过在攻击者控制的页面上覆盖隐藏的帧，迫使用户在受攻击的应用程序中执行非期望的操作(通过限制`frame-ancestors`来保护)

- Mixed content
 
  意外地将资源从不安全的协议中加载到通过HTTPS交付的页面上(通过使用`upgrade-insecure-requests`和`block-all-mixed-content`关键字， 并将脚本和敏感资源的加载限制为HTTPS进行保护)。

因此，只有一小部分CSP指令对XSS保护有用。2.2.2节进一步讨论在应用程序上下文中执行恶意脚本攻击所有指令提供的保护。

#### 2.2.1 采用CSP的好处

### 脚本执行绕过

## CSP实证研究

### 研究问题

### 数据集

### 方法

### 结果与分析

## 改善CSP

## 相关工作

## 结论

## 引用

[1] E. Athanasopoulos, V. Pappas, A. Krithinakis, S. Ligouras, E. P. Markatos, and T. Karagiannis. xjs: practical xss prevention for web application development. In USENIX conference on Web application development, 2010.

[2] A. Barth. Bug 54379 - add basic parser for content security policy, 2011.

[3] A. Barth, D. Veditz, and M. West. Content security policy level 2. W3C Working Draft, 2014.

[4] D. Bates, A. Barth, and C. Jackson. Regular expressions considered harmful in client-side xss filters. WWW ’10.

[5] H. Bojinov, E. Bursztein, and D. Boneh. Xcs: cross channel scripting and its impact on web applications. CCS ’09.

[6] CERT. Advisory ca-2000-02 malicious html tags embedded in client web requests, Feb. 2000.

[7] A. Doup´e, W. Cui, M. Jakubowski, M. Peinado, C. Kruegel, and G. Vigna. dedacota: toward preventing server-side xss via automatic code and data separation. In CCS’13.

[8] M. Foundation. Csp policy directives, 2016.

[9] M. V. Gundy and H. Chen. Noncespaces: Using randomization to enforce information flow tracking and thwart cross-site scripting attacks. In NDSS, 2009.

[10] R. Hansen and J. Grossman. Clickjacking, 2008.

[11] D. Hausknecht, J. Magazinius, and A. Sabelfeld. May i?-content security policy endorsement for browser extensions. In DIMVA’15.

[12] B. Hayak. Same origin method execution (some): Exploiting a callback for same origin policy bypass, 2014.

[13] M. Heiderich, M. Niemietz, F. Schuster, T. Holz, and J. Schwenk. Scriptless attacks: stealing the pie without touching the sill. In CCS’12.

[14] M. Heiderich, J. Schwenk, T. Frosch, J. Magazinius, and E. Z. Yang. mxss attacks: Attacking well-secured web-applications by using innerhtml mutations. In CCS’13.

[15] E. Homakov. Using content-security-policy for evil, 2014.

[16] T. Jim, N. Swamy, and M. Hicks. Defeating script injection attacks with browser-enforced embedded policies. In WWW’07.

[17] M. Johns. Script-templates for the content security policy. Journal of Information Security and Applications, 2014.

[18] N. Jovanovic, C. Kruegel, and E. Kirda. Pixy: A static analysis tool for detecting web application vulnerabilities. In S&P’06.

[19] C. Kerschbaumer, S. Stamm, and S. Brunthaler. Injecting csp for fun and security.

[20] A. Klein. Dom based cross site scripting or xss of the third kind. Web Application Security Consortium Articles 4, 2005.

[21] S. Lekies, B. Stock, and M. Johns. 25 million flows later: large-scale detection of dom-based xss. In CCS’13.

[22] M. T. Louw and V. Venkatakrishnan. Blueprint: Robust prevention of cross-site scripting attacks for existing browsers. In Security and Privacy, 2009. IEEE, 2009.

[23] G. Maone. Noscript.

[24] MITRE. Common vulnerabilities and exposures - the standard for information security vulnerability names.

[25] Y. Nadji, P. Saxena, and D. Song. Document structure integrity: A robust basis for cross-site scripting defense. In NDSS, 2009.

[26] T. Oda, G. Wurster, P. C. van Oorschot, and A. Somayaji. Soma: Mutual approval for included content in web pages. In CCS’08.

[27] K. Patil and B. Frederik. A measurement study of the content security policy on real-world applications. International Journal of Network Security, 2016.

[28] D. Ross. IE 8 xss filter architecture/implementation. Blog: http://goo.gl/eOiPsI, 2008.

[29] P. Saxena, S. Hanna, P. Poosankam, and D. Song. Flax: Systematic discovery of client-side validation vulnerabilities in rich web applications. In NDSS, 2010.

[30] W. Security. Website security statistics report, May 2013.

[31] S. Stamm, B. Sterne, and G. Markham. Reining in the web with content security policy. In WWW’10.

[32] B. Sterne. Creating a safer web with content security policy, 2011.

[33] B. Stock, S. Lekies, T. Mueller, P. Spiegel, and M. Johns. Precise client-side protection against dom-based cross-site scripting. In USENIX Security, 2014.

[34] P. Vogt, F. Nentwich, N. Jovanovic, E. Kirda, C. Kruegel, and G. Vigna. Cross site scripting prevention with dynamic data tainting and static analysis. In NDSS, 2007.

[35] G. Wassermann and Z. Su. Static detection of cross-site scripting vulnerabilities. In ICSE’08.

[36] M. Weissbacher, T. Lauinger, and W. Robertson. Why is csp failing? trends and challenges in csp adoption. In RAID’14.

[37] D. Wichers. Owasp top-10 2013. OWASP Foundation, February, 2013.

[38] M. Zalewski. Postcards from the post-xss world. Online at http://lcamtuf.coredump.cx/postxss, 2011.

[39] M. Zalewski. The subtle / deadly problem with csp. Online at http://goo.gl/sK4w7q, 2011.

    
