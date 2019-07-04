# Beacon

sendBeacon（） -> [sendBeacon]()
时间关键 -> time-critical

[w3c spec地址]()

此规范定义了一个接口，Web开发人员可以使用该接口来调度异步和非阻塞的数据传输，从而最大限度地减少与其他time-critical操作的资源争用，同时确保仍然处理这些请求并将其传递到目标。

## 文档当前状态

本文档由[Web性能工作组]()作为编辑草案发布。

[GitHub issues]()是讨论本规范的首选。或者，您可以将评论发送到我们的邮件列表。请将它们发送到public-web-perf@w3.org（[档案]()）。

作为编辑草案出版并不意味着W3C会员资格的认可。这是一份草案文件，可能随时被其他文件更新，替换或废弃。除了正在进行的工作之外，引用此文档是不恰当的。

本文档由一个根据[W3C专利政策]()运营的小组制作。 W3C维护一份与该集团可交付成果有关的[任何专利披露的公开名单]();该页面还包括披露专利的说明。具有个人认为包含[基本要求]()的专利的实际知识的个人必须根据[W3C专利政策的第6节]()披露该信息。

本文档受2019年3月1日W3C流程文档的约束。

## 1. 介绍

Web应用程序通常需要向一个或多个服务器发出报告事件、状态更新和分析的请求。此类请求通常不需要客户端上的响应处理（例如，HTTP状态码为204或200，响应主体为空的），并且不应与其他高优先级操作竞争网络和计算资源，例如获取关键资源，对输入作出响应，运行动画等。但是，此类单向请求（信标）还负责提供关键应用程序和测量数据，迫使开发人员使用昂贵的方法来确保其交付：

* 开发人员选择立即交付每个信标，而不是合并和推迟交付，因为这样可以提高交付率。 延迟传递可能意味着信标请求可能没有足够的时间来成功完成，这导致重要的应用数据丢失。
* 开发人员选择通过同步XMLHttpRequest发出阻塞请求，插入无操作繁忙循环，或使用阻止用户代理执行time-critical操作（例如，点击，卸载和其他处理程序）并损害用户体验的其他技术。 阻止行为用于提供改进的传递速率，因为它阻止用户代理和操作系统在系统卸载，暂停或终止页面时取消请求。

上述交付和处理要求之间的不匹配使大多数开发人员难以选择并广泛采用阻碍用户体验的阻止技术。 此规范定义了一个接口，Web开发人员可以使用该接口来调度异步和非阻塞的数据传输，从而最大限度地减少与其他time-critical操作的资源争用，同时确保仍然处理这些请求并将其传递到目标：

* 给信标请求设置较低优先级，以避免与time-critical型操作和更高优先级的网络请求竞争。
* 用户代理可以有效地合并信标请求以优化移动设备上的能量使用。
* 保证在卸载页面之前启动信标请求，并且允许信标请求运行完成，而不需要阻塞请求或阻止用户交互事件处理的其他技术。

以下示例显示了使用[sendBeacon()]()方法发送事件，单击和分析数据：

[EXAMPLE 1](https://w3c.github.io/beacon/#example-1)
```html
<html>
<script>
  // emit non-blocking beacon to record client-side event
  function reportEvent(event) {
    var data = JSON.stringify({
      event: event,
      time: performance.now()
    });
    navigator.sendBeacon('/collector', data);
  }

  // emit non-blocking beacon with session analytics as the page
  // transitions to background state (Page Visibility API)
  document.addEventListener('visibilitychange', function() {
    if (document.visibilityState === 'hidden') {
      var sessionData = buildSessionReport();
      navigator.sendBeacon('/collector', sessionData);
    }
  });
</script>

<body>
 <a href='http://www.w3.org/' onclick='reportEvent(this)'>
 <button onclick="reportEvent('some event')">Click me</button>
</body>
</html>
```

> 上面的示例使用[[PAGE-VISIBILITY-2]]()中定义的[visibilitychange]()事件来触发会话数据的传递。 当页面转换到后台状态时（例如，当用户切换到不同的应用程序，进入主屏幕等）或正在卸载时，此事件是唯一保证在移动设备上触发的事件。开发人员应该避免依赖卸载事件，因为只要页面处于后台状态（即visibilityState等于隐藏），它就不会触发，并且进程由移动操作系统终止。

通过[sendBeacon()]()方法发起的请求不会阻止或与time-critical的工作竞争，可以由用户代理优先化以提高网络效率，并且消除使用阻塞操作来确保信标数据的传递的需要。

sendBeacon（）不会做什么，也不打算解决：

* sendBeacon（）方法不为脱机存储或传递提供特殊处理。 信标请求与任何其他请求一样，可以与[[SERVICE-WORKERS]]()结合使用，以在必要时提供离线功能。
* sendBeacon（）方法不用于提供后台同步或传输功能。 用户代理限制最大可接受的有效负载大小，以确保信标请求能够快速且及时地完成。
* sendBeacon（）方法不提供自定义请求方法，提供自定义请求标头或更改请求和响应的其他[处理属性]()的功能。 需要对此类请求进行非默认设置的应用程序应使用[[FETCH]] API并将[keep-alive]()标志设置为`true`。

## 2. 一致性要求

本规范中的所有图表，示例和注释都是非规范性的，所有明确标记为非规范性的部分也是如此。本规范中的其他所有内容都是规范性的。

在本文件的规范部分中，关键词“必须”，“必须”，“不需要”，“应该”，“不应该”，“推荐”，“可以”和“可选”应该被解释为描述在[[RFC2119]]()中。为了便于阅读，这些单词在本规范中并未以全部大写字母出现。

作为算法的一部分在命令中表达的要求（例如“剥离任何前导空格字符”或“返回错误并中止这些步骤”）将被解释为关键词的意思（“必须”，“应该”，“可能“等”用于引入算法。

一些一致性要求被表述为对属性，方法或对象的要求。这些要求应被解释为对用户代理的要求。

只要最终结果是等效的，可以以任何方式实现表述为算法或特定步骤的一致性要求。 （特别是，本说明书中定义的算法旨在易于遵循，而不是为了保持高效。）

### 2.1 依赖(TODO 外链)

#### DOM
The following terms are defined in the DOM specification: [DOM]

* the document URL
* document

#### HTML
The following terms are defined in the HTML specification: [HTML]

* API base URL
* entry settings object
* multipart/form-data boundary string
* multipart/form-data encoding algorithm
* resource origin
* the Navigator object

#### Fetch
The following terms are defined in the HTML specification: [FETCH]

* header value
* request
* request method
* request url
* request header list
* request origin
* request keep-alive flag
* request body
* request mode
* request credentials mode
* fetch
* http-network-or-cache-fetch
* BodyInit
* CORS-safelisted request-header
* Extract
* the Access-Control-Allow-Credentials header
* the Access-Control-Allow-Origin header
* the Access-Control-Allow-Headers header

#### URL
The following terms are defined in the URL specification: [URL]

* URL parser
* scheme

#### Web IDL
The IDL fragments in this specification must be interpreted as required for conforming IDL fragments, as described in the Web IDL specification [WEBIDL].

The following terms are defined in the Web IDL specification:

* throw
* USVString type
* TypeError type

#### Page Visibility
The following term is defined in the Page Visibility specification: [PAGE-VISIBILITY-2]

* visibilitychange
* unload
* hidden
* visibilityState

## 3. Beacon

### 3.1 sendBeacon（）方法

```WebIDL
partial interface Navigator {
    boolean sendBeacon(USVString url, optional BodyInit? data = null);
};
```

sendBeacon（）方法将data参数提供的数据传输到url参数提供的URL：

* 用户代理必须在设置了[keepalive]()标志的情况下启动提取，这限制了这些请求可以排队的数据量，以确保信标请求能够快速及时地完成。
* 当文档[visibilityState]()转换为[hidden]()时，用户代理必须安排所有信标请求的立即传输，并且必须允许所有这些请求运行完成而不阻止其他时间关键和高优先级工作。
* 用户代理应该安排提供数据的传输，以最小化与其他时间关键和高优先级工作的资源（CPU和网络）争用。
* 用户代理可以延迟所提供数据的传输以优化网络和能量效率 - 例如， 如果网络处于活动状态，则立即传送，或等到网络接口处于活动状态。 但是，用户代理不应无限期地延迟传输，并确保即使没有其他网络活动也会定期刷新挂起的传输。

> Beacon API不提供响应回调。 建议服务器省略为此类请求返回响应正文（例如，使用204 No Content响应）。

#### 参数

##### url

[url]()参数指示要传输数据的URL。

##### data

[data]参数是要传输的[BodyInit]()数据。

#### 返回值

如果用户代理能够成功排队数据以进行传输，则sendBeacon（）方法返回true。 否则返回false。

> 用户代理对可以通过此API发送的数据量施加限制：这有助于确保成功传递此类请求，并且对其他用户和浏览器活动的影响最小。 如果要排队的数据量超过用户代理限制（如[http-network-or-cache-fetch]()中所定义），则此方法返回false; 返回值为true表示浏览器已将数据排队等待传输。 但是，由于实际数据传输是异步发生的，因此该方法不提供数据传输是否成功的任何信息。

### 3.2 处理模型

[sendBeacon]: https://w3c.github.io/beacon/#dom-navigator-sendbeacon
[\[PAGE-VISIBILITY-2\]]: https://w3c.github.io/beacon/#bib-page-visibility-2
[visibilitychange]: https://www.w3.org/TR/page-visibility-2/#dfn-visibilitychange
[Web性能工作组]: https://www.w3.org/webperf/
[档案]: https://lists.w3.org/Archives/Public/public-web-perf/
[GitHub issues]: https://github.com/w3c/beacon/issues/
[w3c spec地址]: https://w3c.github.io/beacon/#sec-sendBeacon-method
[W3C专利政策]: https://www.w3.org/Consortium/Patent-Policy/
[W3C专利政策的第6节]: https://www.w3.org/Consortium/Patent-Policy/#sec-Disclosure
[任何专利披露的公开名单]: https://www.w3.org/2004/01/pp-impl/45211/status
[基本要求]: https://www.w3.org/Consortium/Patent-Policy/#def-essential
[\[SERVICE-WORKERS\]]: https://w3c.github.io/beacon/#bib-service-workers
[处理属性]: https://w3c.github.io/beacon/#sec-processing-model
[\[FETCH\]]: https://w3c.github.io/beacon/#bib-fetch
[keep-alive]: https://fetch.spec.whatwg.org/#keep-alive-flag
[\[RFC2119\]]: https://w3c.github.io/beacon/#bib-rfc2119
[keepalive]: https://w3c.github.io/beacon/#concept-keep-alive-flag
[hidden]: https://www.w3.org/TR/page-visibility-2/#dom-visibilitystate-hidden
[visibilityState]: https://www.w3.org/TR/page-visibility-2/#dom-visibilitystate
[url]: https://w3c.github.io/beacon/#url-parameter
[data]: https://w3c.github.io/beacon/#dfn-data
[BodyInit]: https://fetch.spec.whatwg.org/#bodyinit
[http-network-or-cache-fetch]: https://fetch.spec.whatwg.org/#http-network-or-cache-fetch