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

> '拥有 dispatch 方法的组件为 redux 中的 container component'
