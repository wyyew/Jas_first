### redux 中的同步的数据流，流程如下：

view -> actionCreator -> action -> reducer -> newState -> container component 

### redux 中的异步数据流，流程如下：

view -> asyncActionCreator -> wait -> action -> reducer -> newState -> container component

## 通过 middleware 实现异步

我们已经很清楚一个 middleware 的结构 ，其核心的部分为

```
function(action) {
    // 调用后面的 middleware
    next(action)
}
```

middleware 完全掌控了 reducer 的触发时机， 也就是 action 到了这里完全由中间件控制，不乐意就不给其他中间件处理的机会，而且还可以控制调用其他中间件的时机。

> 同步调用和异步调用的方式不相同:

- 同步的情况: store.dispatch(actionCreator(payload))

- 异步的情况: asyncActionCreator(store.dispatch, payload)

幸运的是在 redux 中通过 middleware 机制可以很容易的解决上面的问题

## 通过 middleware 实现异步 

我们已经很清楚一个 middleware 的结构 ，其核心的部分为

```
function (action) {
    // async call 
    fetch('....')
      .then(
          function resolver(ret) {
            newAction = createNewAction(ret, action)
            next(newAction)
          },
          function rejector(err) {
            rejectAction = createRejectAction(err, action)
            next(rejectAction)
          })
    });
}
```

任何异步的 javascript 逻辑都可以，如： ajax callback, Promise, setTimeout 等等, 也可以使用 es7 的 async 和 await。

### 第三方异步组件

- redux-thunk

- redux-promise

- redux-saga

## redux-thunk 介绍

> 简单来说一个 thunk 就是一个封装表达式的函数，封装的目的是延迟执行表达式

```
// 1 + 2 立即被计算 = 3
let x = 1 + 2;

// 1 + 2 被封装在了 foo 函数内
// foo 可以被延迟执行
// foo 就是一个 thunk 
let foo = () => 1 + 2;

```

redux-thunk 是一个通用的解决方案，其核心思想是让 action 可以变为一个 thunk ，这样的话:

- 同步情况：dispatch(action)

- 异步情况：dispatch(thunk)

我们已经知道了 thunk 本质上就是一个函数，函数的参数为 dispatch, 所以一个简单的 thunk 异步代码就是如下：

```
this.dispatch(function (dispatch){
    setTimeout(() => {
       dispatch({type: 'THUNK_ACTION'}) 
    }, 1000)
})
```

之前已经讲过，这样的设计会导致异步逻辑放在了组件中，解决办法为抽象出一个 asyncActionCreator, 这里也一样，我们就叫 thunkActionCreator 吧，
上面的例子可以改为:

```
//actions/someThunkAction.js
export function createThunkAction(payload) {
    return function(dispatch) {
        setTimeout(() => {
           dispatch({type: 'THUNK_ACTION', payload: payload}) 
        }, 1000)
    }
}

// someComponent.js
this.dispatch(createThunkAction(payload))
```
### 安装和使用

第一步：安装

```
$ npm install redux-thunk
```

第二步: 添加 thunk 中间件

```
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers/index';

const store = createStore(
  rootReducer,
  applyMiddleware(thunk)
);
```

第三步：实现一个 thunkActionCreator

```
//actions/someThunkAction.js
export function createThunkAction(payload) {
    return function(dispatch) {
        setTimeout(() => {
           dispatch({type: 'THUNK_ACTION', payload: payload}) 
        }, 1000)
    }
}
```

第三步：组件中 dispatch thunk

```
this.dispatch(createThunkAction(payload));
```

> ** 拥有 dispatch 方法的组件为 redux 中的 container component **

## thunk 源码

说了这么多，redux-thunk 是不是做了很多工作，实现起来很复杂，那我们来看看 thunk 中间件的实现

```
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

就这么简单，只有 14 行源码，但是这简短的实现却能完成复杂的异步处理，怎么做到的，我们来分析一下：

判断如果 action 是 function 那么执行 action(dispatch, getState, ...)

action 也就是一个 thunk

执行 action 相当于执行了异步逻辑

action 中执行 dispatch

开始新的 redux 数据流，重新回到最开始的逻辑（thunk 可以嵌套的原因）

把执行的结果作为返回值直接返回

直接返回并没有调用其他中间件，也就意味着中间件的执行在这里停止了

可以对返回值做处理（后面会讲如果返回值是 Promise 的情况）

如果不是函数直接调用其他中间件并返回

理解了这个过后是不是对 redux-thunk 的使用思路变得清晰了

## thunk 的组合

根据 redux-thunk 的特性，可以做出很有意思的事情

1. 可以递归的 dispatch(thunk) -> 实现 thunk 的组合；

2. thunk 运行结果会作为 dispatch返回值 -> 利用返回值为 Promise 可以实现多个 thunk 的编排;

thunk 组合例子：

```
function thunkC() {
    return function(dispatch) {
        dispatch(thunkB())
    }
}
function thunkB() {
    return function (dispatch) {
        dispatch(thunkA())
    }
}
function thunkA() {
    return function (dispatch) {
        dispatch({type: 'THUNK_ACTION'})
    }
}
```

Promise 例子

```
function ajaxCall() {
    return fetch(...);
}

function thunkC() {
    return function(dispatch) {
        dispatch(thunkB(...))
        .then(
            data => dispatch(thunkA(data)),
            err  => dispatch(thunkA(err))
        )
    }
}
function thunkB() {
    return function (dispatch) {
        return ajaxCall(...)
    }
}

function thunkA() {
    return function (dispatch) {
        dispatch({type: 'THUNK_ACTION'})
    }
}

```

## redux-promise

另外一个 redux 文档中提到的异步组件为 redux-promise, 我们直接分析一下其源码吧

```
import { isFSA } from 'flux-standard-action';

function isPromise(val) {
  return val && typeof val.then === 'function';
}

export default function promiseMiddleware({ dispatch }) {
  return next => action => {
    if (!isFSA(action)) {
      return isPromise(action)
        ? action.then(dispatch)
        : next(action);
    }

    return isPromise(action.payload)
      ? action.payload.then(
          result => dispatch({ ...action, payload: result }),
          error => {
            dispatch({ ...action, payload: error, error: true });
            return Promise.reject(error);
          }
        )
      : next(action);
  };
}
```
大概的逻辑就是：

如果不是标准的 flux action，那么判断是否是 promise, 是执行 action.then(dispatch)，否执行 next(action)

如果是标准的 flux action, 判断 payload 是否是 promise，是的话 payload.then 获取数据，然后把数据作为 payload 重新 dispatch({ ...action, payload: result}) , 否执行 next(action)

结合 redux-promise 可以利用 es7 的 async 和 await 语法，简化异步的 promiseActionCreator 的设计， eg:

```
export default async (payload) => {
  const result = await somePromise;
  return {
    type: "PROMISE_ACTION",
    payload: result.someValue;
  }
}
```

