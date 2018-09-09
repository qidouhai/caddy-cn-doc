---
title: "HTTPS劫持诊断"
sitename: "Caddy中文文档"
---

# HTTPS劫持诊断

Caddy有能力检测是否有中间人(MITM)在攻击HTTPS连接，虽然浏览器和最终用户可能看不到这些攻击。这意味着Caddy可以确定TLS代理是否“可能”或“不可能”正在积极地拦截HTTPS连接。

<iframe src="/mitm/check" frameborder="0" width="100%" height="200"></iframe>

根据Durumeric, Halderman等人在他们的[NDSS '17 paper](https://jhalderm.com/pub/papers/interception-ndss17.pdf)中描述的技术，所有传入的HTTPS连接都会自动检查是否被篡改。检查结果在通过各种方式传达给Caddy，因此你可以选择如何处理可疑的MITM攻击您的客户。(请记住，许多TLS代理以“仁慈的”反病毒或防火墙产品的形式出现。)

被拦截的TLS连接不安全，尽管软件供应商的广告与此相反。对可疑的MITM攻击的反响程度取决于你自己、站点的性质、受众以及政治环境。您可能会以以下任何一种方式对可疑的HTTPS拦截做出响应(为了增加炫耀性):

* 使用{mitm}[占位符](placeholder.md)和自定义日志格式把发生了什么记录到[日志](log.md)。

* 如果代理一个上游应用程序，在`proxy`指令中使用header_upstream[添加一个请求头](proxy.md)，其值包含{mitm}。

* 如果您正在使用[模板](templates.md)，请使用`{{.IsMITM}}`这个[模板操作](template-actions.md)在您的网站上向用户显示警告。

* 将URI通过[内部重写](rewrite.md)到展示专用警告、错误或信息的页面。

* 将用户[重定向](redir.md)到另一个页面或站点。

* 立即关闭连接返回一个空响应(可能很快就会出现哦)。

在执行任何可能被认为是极端的措施之前，请阅读完整个页面。


## 免责声明

Caddy的作者、维护人员和贡献者将真诚地尝试让这个特性在主流浏览器的常用版本中正确工作，但不能保证绝对的准确性。这个特性依赖于硬编码的试探策略，这种方法试图通过TLS握手来识别浏览器。浏览器和操作系统更新可能会在任何时候使试探策略过时。Caddy开发人员不应对使用此特性可能导致的任何损害、成本、错误通信或误解或其他后果负责。请理智使用，自负风险。

## 支持客户

Caddy被设计用来保护Chrome、Firefox、IE/Edge和Safari的最新版本。这些浏览器的最新开发版本可能还没有被识别(如果没有，请告诉我们!)。我们也正在实验性地尝试识别和支持Tor浏览器。


## 错误报警

Caddy有时会错误地将连接标记为“可能”被拦截，即使它不是。这通常发生在客户端使用了篡改过的User-Agent字符串时。为了得到最好的保护，我们建议用户不要改变他们的User-Agent头信息，并且站点所有者要保持Caddy的更新。

也有可能一些浏览器/平台的组合还没有被考虑到。要报告错误警告，请用您真实未经修改的User-Agent字符串、浏览器版本、操作系统/平台详细信息、原始ClientHello字节以及任何其他相关构建信息[提交问题](https://github.com/mholt/caddy/issues/new)。您还必须确保您的连接是在一个可信的网络上建立的，该网络没有防火墙或代理，并且所有操作系统“安全”产品在您的计算机和本地网络上都完全禁用。(你必须说服我们，这个连接实际上是安全的，我们必须能够重现你报告的问题。)

## 漏报

当HTTPS拦截发生时，Caddy有检测不到的时候(“unlikely”分类)，原因可能很多，从简单的到最坏的都有:

* 这可能仅仅是因为Caddy的检测试探策略不够全面。请[提交一个问题](https://github.com/mholt/caddy/issues/new)，最好能提供足够的信息用来捕获拦截，包括ClientHello字节和许多有关MITM软件的详细信息。

* 客户端没有发送可识别的User-Agent头。除了少数例外，Caddy仅在主流浏览器上检查MITM。

* TLS代理正在保留客户端和服务器之间TLS握手的原始属性。这种情况不是最糟糕的漏报，因为如果TLS连接很弱，至少浏览器能够显示警告。

* TLS代理正在修改或去除User-Agent头，可能是为了隐藏。然而，任何修改HTTP请求的TLS代理都有破坏HTTP的风险，这会暴露它的存在(类似于重力暴露黑洞的方式)。

Caddy的MITM检测功能之所以能够正常工作，主要是因为TLS代理的实现很草率，文档很少，而且很少更新。