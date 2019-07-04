# Beacon

sendBeacon（） -> [sendBeacon]()
时间关键 -> time-critical

[w3c spec地址](https://w3c.github.io/beacon/#sec-sendBeacon-method)

此规范定义了一个接口，Web开发人员可以使用该接口来调度异步和非阻塞的数据传输，从而最大限度地减少与其他time-critical操作的资源争用，同时确保仍然处理这些请求并将其传递到目标。

## 文档当前状态

本文档由[Web性能工作组](https://www.w3.org/webperf/)作为编辑草案发布。

[GitHub issues](https://github.com/w3c/beacon/issues/)是讨论本规范的首选。或者，您可以将评论发送到我们的邮件列表。请将它们发送到public-web-perf@w3.org（[档案](https://lists.w3.org/Archives/Public/public-web-perf/)）。

作为编辑草案出版并不意味着W3C会员资格的认可。这是一份草案文件，可能随时被其他文件更新，替换或废弃。除了正在进行的工作之外，引用此文档是不恰当的。

本文档由一个根据[W3C专利政策](https://www.w3.org/Consortium/Patent-Policy/)运营的小组制作。 W3C维护一份与该集团可交付成果有关的[任何专利披露的公开名单](https://www.w3.org/2004/01/pp-impl/45211/status);该页面还包括披露专利的说明。具有个人认为包含[基本要求](https://www.w3.org/Consortium/Patent-Policy/#def-essential)的专利的实际知识的个人必须根据[W3C专利政策的第6节](https://www.w3.org/Consortium/Patent-Policy/#sec-Disclosure)披露该信息。

本文档受2019年3月1日W3C流程文档的约束。

## 1 介绍

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

> 上面的示例使用[[PAGE-VISIBILITY-2]]()中定义的[visibilitychange](https://www.w3.org/TR/page-visibility-2/#dfn-visibilitychange)事件来触发会话数据的传递。 当页面转换到后台状态时（例如，当用户切换到不同的应用程序，进入主屏幕等）或正在卸载时，此事件是唯一保证在移动设备上触发的事件。开发人员应该避免依赖卸载事件，因为只要页面处于后台状态（即visibilityState等于隐藏），它就不会触发，并且进程由移动操作系统终止。

通过[sendBeacon()]()方法发起的请求不会阻止或与time-critical的工作竞争，可以由用户代理优先化以提高网络效率，并且消除使用阻塞操作来确保信标数据的传递的需要。

sendBeacon（）不会做什么，也不打算解决：

* sendBeacon（）方法不为脱机存储或传递提供特殊处理。 信标请求与任何其他请求一样，可以与[[SERVICE-WORKERS]](https://w3c.github.io/beacon/#bib-service-workers)结合使用，以在必要时提供离线功能。

[sendBeacon]: https://w3c.github.io/beacon/#dom-navigator-sendbeacon
[\[PAGE-VISIBILITY-2\]]: https://w3c.github.io/beacon/#bib-page-visibility-2
