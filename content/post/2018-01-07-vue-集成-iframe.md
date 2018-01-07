---
title: vue 集成 iframe
date: 2018-01-07T14:09:22+08:00
tags:
- web
- vue
- iframe
---

书接上回：最近不是在用 [vue](https://vuejs.org/) 开发 H5 App 嘛（再后来又加上了 react-native 😂，这个以后有机会说），我们有个接口返回的是 HTML 页面，在原来的原生 App 里面自然是用个 WebView 控件来展示啦，但我们现在整个应用都是 H5 了，HTML 里面嵌另一个 HTML，那第一反应自然就是 iframe 啰。

PS. H5 里面是嵌不了原生控件的对吧？react-native 是可以的，有时确实是要方便一些的……吧

PSS. 除了 iframe 之外呢，最近好像还有一个比较新的标准是 [web components](https://www.webcomponents.org/)，也是用来做内嵌的吧，但对浏览器的版本要求应该比较高（虽然也是有 polyfill 的啦），记得我当时好像也是试了一下的，但是没成功 😂，也不记得是啥问题了，而且 iframe 也能满足要求啦，所以就没弄这个了。回头有机会再研究下吧。

如果只是简单的页面展示的话，直接通过 `<iframe>` 控件的 `:src` 属性绑定页面 URL 就完事啦。

但我们这是个 App 嘛，页面里面也是有一些可交互元素的，比如点击跳转到详情页之类的，这时就要绑定 `<iframe>` 的 `@load` 事件（vue 写法，对应于 JS 接口里面的 `onload` 事件），示例代码如下：

<script src="https://gist.github.com/glacjay/3c320321b1709454fdb7e06e26b0132c.js"></script>

要点：

- 第 15 行：在 `@load` 事件回调方法 `onIframeLoad` 中，iframe 控件本身可以从 `event.target` 中得到，不需要用 vue 的 ref 之类的。
- 第 16 行：要从 iframe 页面中访问外部提供的接口，可以通过发送 `message` 事件来实现，那么在外部的 HTML 环境中就要通过 [`window.addEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) 来设置对该事件的监听函数。这里的 `window` 对象就是 iframe 外部 vue 所在的浏览器对象。
- 第 17 行：从外部访问 iframe 本身的 `window` 对象就是通过 `iframe.contentWindow` 来得到，而调用该对象的 `eval` 方法就是从外部对 iframe 进行操作的一般方法了。该方法接受一个字符串参数，可以传入一段完整的 JS 代码，这时用 JS 新特性 template literals 来传这个参数就会很舒服了，既支持多行，又支持内嵌表达式，可以传点儿参数进去。
- 第 18 行：耶，服务端返回回来的页面自带 ~~jQuery~~ [zepto](http://zeptojs.com/)，这让我们的字符串小脚本好写很多啦。不过要注意：由于这段脚本是要直接在 iframe 所在的浏览器环境（WebView）里面跑的，而不是像外面的 vue 会先经过 babel 的编译（因为用了 [.vue 格式](https://vuejs.org/v2/guide/single-file-components.html)嘛），所以最好不要用那些个比较新的语法，比如这里就没有用 `() => {}` 的匿名函数写法（因为……我也懒得查啦，而且手机的 WebView 兼容性嘛，你懂的 😏）。
- 第 19 行：在 iframe 环境中，`window.parent` 就是其外部环境的 `window` 对象啦，调用其 [`postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) 方法就可以往外边发 `message` 事件了，所传参数会最终变成该事件的 `data` 字段。
- 第 30 行：最终接收到 iframe 传出来的 `message` 事件，然后就……该干嘛干嘛啰。

OK，以上就是本次的 vue 集成 iframe 并实现互操作的方法总结啦。
