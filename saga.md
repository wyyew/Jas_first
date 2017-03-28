## redux-saga 介绍

[](https://zhuanlan.zhihu.com/p/21398212)

redux-saga 也是解决 redux 异步 action 的一个中间件，不过和之前的设计有本质的不同

1. redux-saga 完全基于 Es6 的 Generator Function

2. 不使用 actionCreator 策略，而是通过监控 action, 然后在自动做处理

3. 所有带副作用的操作（异步代码， 业务控制逻辑代码）都被放到 saga 中

## 到底什么是 saga

> The term saga is commonly used in discussions of CQRS to refer to a piece of code that coordinates and routes messages between bounded contexts and aggregates.

这个定义的核心就是 CQRS-查询与责任分离 ，对应到 redux-sage 就是 action 与 处理函数的分离。 实际上在 redux-saga 中，一个 saga 就是一个 Generator 函数。

eg:

```
import { takeEvery, takeLatest } from 'redux-saga'
import { call, put } from 'redux-saga/effects'
import Api from '...'

/*
 * 一个 saga 就是一个 Generator Function 
 *
 * 每当 store.dispatch `USER_FETCH_REQUESTED` action 的时候都会调用 fetchUser.
 */
function* mySaga() {
  yield* takeEvery("USER_FETCH_REQUESTED", fetchUser);
}

/**
 * worker saga： 真正处理 action 的 saga
 *  
 * USER_FETCH_REQUESTED action 触发时被调用
 * @param {[type]} action  [description]
 * @yield {[type]} [description]
 */
function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}
```
## 一些基本概念

- watcher saga

负责编排和派发任务的 saga

- worker saga

真正负责处理 action 的函数

- saga helper

如上面例子中的 takeEvery，简单理解就是用于监控 action 并派发 action 到 worker saga 的辅助函数

- Effect

redux-saga 完全基于 Generator 构建，saga 逻辑的表达是通过 ** yield javascript 对象来实现，这些对象就是Effects **。

这些对象相当于描述任务的规范化数据（任务如执行异步函数，dispatch action 到一个 store），这些数据被发送到 redux-saga 中间件中执行，如：

1. put({type: "USER_FETCH_SUCCEEDED", user: user}) 表示要执行 dispatch({{type: "USER_FETCH_SUCCEEDED", user: user}}) 任务

2. call(fetch, url) 表示要执行 fetch(url)

通过这种 effect 的抽象，可以避免 call 和 dispatch 的立即执行，而是描述要执行什么任务，这样的话就很容易对 saga 进行测试，saga 所做的事情就是将这些 effect 编排起来用于描述任务，真正的执行都会放在 middleware 中执行。

## 安装和使用

### 第一步：安装

```
$ npm install --save redux-saga
```

### 第二步：添加 saga 中间件

```
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import reducer from './reducers'
import mySaga from './sagas'

// 创建 saga 中间件
const sagaMiddleware = createSagaMiddleware()

// 添加到中间件中
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)

// 立即运行 saga ，让监控器开始监控
sagaMiddleware.run(mySaga)
```

