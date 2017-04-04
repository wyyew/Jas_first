# react-router

## react-router 中的history
> history 一个管理js应用session会话历史的js库。它将不同环境（浏览器，node...）的变量统一成了一个简易的API来管理历史堆栈、导航、确认跳转、以及sessions间的持续状态。

```
//基本 使用
import { createHistory } from 'history'

const history = createHistory()

// 当前的地址
const location = history.getCurrentLocation()

// 监听当前的地址变换
const unlisten = history.listen(location => {
  console.log(location.pathname)
})

// 将新入口放入历史堆栈
history.push({
  pathname: '/the/path',
  search: '?a=query',

  // 一些不存在url参数上面的当前url的状态值
  state: { the: 'state' }
})

// When you're finished, stop the listener
unlisten()
```

你也可以使用 history对象来的方法来改变当前的location：

- push(location)
- replace(location)
- go(n)
- goBack()
- goForward()

一个 history 知道如何去监听浏览器地址栏的变化， 并解析这个 URL 转化为 location 对象， 然后 router 使用它匹配到路由，最后正确地渲染对应的组件。

location对象包括：

- pathname      同window.location.pathname
- search        同window.location.search
- state         一个捆绑在这个地址上的object对象
- action        PUSH, REPLACE, 或者 POP中的一个
- key           唯一ID

> 使用 hashHistory，浏览器上看到的 url 会是这样的: /#/user/haishanh?_k=adseis
使用 browserHistory，浏览器上看到的 url 会是这样的：/user/haishanh
看起来当然 browserHistory 很好很理想，但 browserHistory 需要 server 端支持。 而使用hashHistory的时候，因为 url 中 # 符号的存在，从 /#/ 到 /#/user/haishanh 浏览器并不会去发送一次 request，react-router 自己根据 url 去 render 相应的模块。
而使用 browserHistory 的时候，浏览器从 / 到 /user/haishanh 是会向 server 发送 request 的。所以 server 端是要做特殊配置的。比如用的 express 的话，你需要 handle 所有的路由 app.get('*', (req, res) => { ... })，使用了 nginx 的话，nginx也要做相应的配置。 具体见这里59，还有这个例子42。
所以你的 App 是静态，没有服务端的话，只能用 hashHistory。

## 改变路由

- browserHistory.push
- this.context.router.push()
