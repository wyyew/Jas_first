## redux-saga简介

> Redux-saga是Redux的一个中间件，主要集中处理react架构中的异步处理工作，被定义为generator(ES6)的形式，采用监听的形式进行工作。

## redux-saga安装

使用npm进行安装：

```
npm install --save redux-saga
```

## redux-saga常用方法解释

1. redux Effects

>Effect 是一个 JavaScript 对象，可以通过 yield 传达给 sagaMiddleware 进行执行在， 如果我们应用redux-saga，所有的 Effect 都必须被 yield 才会执行。

举个例子，我们要改写下面这行代码：
```
yield fetch(url);
```

应用saga:

```
yield call(fatch, url)
```

2. take

等待 dispatch 匹配某个 action 。

比如下面这个例子：

```
....
while (true) {
  yield take('CLICK_Action');
  yield fork(clickButtonSaga);
}
....
```
3. put

触发某个action， 作用和dispatch相同：

```
yield put({ type: 'CLICK' });
```

具体的例子：

```
import { call, put } from 'redux-saga/effects'

export function* fetchData(action) {
   try {
      const data = yield call(Api.fetchUser, action.payload.url)
      yield put({type: "FETCH_SUCCEEDED", data})
   } catch (error) {
      yield put({type: "FETCH_FAILED", error})
   }
}
```
