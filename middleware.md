## Middleware

> **它提供的是位于 action 被发起之后，到达 reducer 之前的扩展点**

你可以利用 Redux middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。

```
//第一种写法
const store = createStore(Counter, applyMiddleware( createLogger() ))
//第二种写法
const finalCreatStore = applyMiddleware( createLogger())(createStore)
const store = finalCreatStore( Counter )
```

## redux-devtools redux-devtools-log-monitor redux-dev-tools-dock-monitor 一个调试面板

## react-router-redux 将路由信息和 store绑定
