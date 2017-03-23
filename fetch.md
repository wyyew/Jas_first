> 先看一下 Fetch 原生支持率：

原生支持率并不高，幸运的是，引入下面这些 polyfill 后可以完美支持 IE8+ ：

- 由于 IE8 是 ES3，需要引入 ES5 的 polyfill: es5-shim, es5-sham
- 引入 Promise 的 polyfill: es6-promise
- 引入 fetch 探测库：fetch-detector
- 引入 fetch 的 polyfill: fetch-ie8
- 可选：如果你还使用了 jsonp，引入 fetch-jsonp
- 可选：开启 Babel 的 runtime 模式，现在就使用 async/await

Fetch polyfill 的基本原理是探测是否存在 window.fetch 方法，如果没有则用 XHR 实现。这也是 github/fetch 的做法，但是有些浏览器（Chrome 45）原生支持 Fetch，但响应中有中文时会乱码，老外又不太关心这种问题，所以我自己才封装了 fetch-detector 和 fetch-ie8 只在浏览器稳定支持 Fetch 情况下才使用原生 Fetch。这些库现在每天有几千万个请求都在使用，绝对靠谱！

终于，引用了这一堆 polyfill 后，可以愉快地使用 Fetch 了。但要小心，下面有坑：

## Fetch 常见坑

- Fetch 请求默认是不带 cookie 的，需要设置 fetch(url, {credentials: 'include'})
- 服务器返回 400，500 错误码时并不会 reject，只有网络错误这些导致请求不能完成时，fetch 才会被 reject。

## IE 使用策略

所有版本的 IE 均不支持原生 Fetch，fetch-ie8 会自动使用 XHR 做 polyfill。但在跨域时有个问题需要处理。

IE8, 9 的 XHR 不支持 CORS 跨域，虽然提供 XDomainRequest，但这个东西就是玩具，不支持传 Cookie！如果接口需要权限验证，还是乖乖地使用 jsonp 吧，推荐使用 [fetch-jsonp](https://github.com/camsong/fetch-jsonp)。如果有问题直接提 issue，我会第一时间解决。

## 标准 Promise 的不足

> 由于 Fetch 是典型的异步场景，所以大部分遇到的问题不是 Fetch 的，其实是 Promise 的。ES6 的 Promise 是基于 Promises/A+ 标准，为了保持简单简洁，只提供极简的几个 API。如果你用过一些牛 X 的异步库，如 jQuery(不要笑) 、Q.js 或者 RSVP.js，可能会感觉 Promise 功能太少了。

## 没有 Deferred
> Deferred 可以在创建 Promise 时可以减少一层嵌套，还有就是跨方法使用时很方便。
ECMAScript 11 年就有过 Deferred 提案，但后来没被接受。其实用 Promise 不到十行代码就能实现 Deferred：es6-deferred。现在有了 async/await，generator/yield 后，deferred 就没有使用价值了。

## 没有获取状态方法：isRejected，isResolved

