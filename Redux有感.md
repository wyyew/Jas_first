[](http://react-china.org/t/redux/2687/4)

1 & 2. 用 Middleware 處理異步 API 請求，支持同時多個請求

一般用一個 Middleware 處理所有 Promise，轉換成幾個中間狀態的 Actions。

實現可以參照這個：[APIClient.js](https://github.com/rixtox/solar/blob/e6cebc838730b121458d7e05b6de53328244919a/src/utils/APIClient.js#L20-L54)

使用的時候只需要構造一個帶 promise(client) -> Promise 和 types[REQUEST, SUCCESS, FAILURE] 字段的 Action 觸發即可。

```
{
  promise(client) {
    return Promise.all([
      client.get('method_1'),
      client.post('method_2')
    ])
  },
  types: ['ACTION', 'ACTION_SUCCESS', 'ACTION_FAIL']
}
```

這個是 Erik Rasmussen 提出的 Ducks102 解決方案，個人覺得這個方法完全符合語義，也正確地發揮了 Middleware 應有的作用。


## 關於你提到的第一個缺點，其實我覺得應該是當做優點來看待的。Redux 其實更加注重數據的單一流向性，所有的 Component 都能變成沒有 states 的 Dumb Component38，這一點反而是 Flux 沒有完全做到的。因此 Redux 能做到很多 Flux 做不到的事情，比如保留 store 的歷史記錄、回放、修改、與後端進行完全同步等等。

關於你說的第二個缺點，我在開發環境下用到 redux-form41 加上 redux-logger16 和 redux-devtools20 這些 Middlewares 的時候，很明顯的感受到了卡頓。所以我是認同你這個觀點的，使用 Redux 這種狀態機模型的狀態管理方式確實會降低效率。不過我覺得這個是可以改善的。我能想到的解決辦法就是使用 Immutable.js29 儲存 store 裡面的數據，在每個大的組件的 componentShouldUpdate 處做一個類似 Caching 的檢查，阻止沒有數據變化的組件樹的更新。但是這樣做的話就得要求盡量少用 react-redux 的 connect 功能，因為這個東西其實等同於 Flux 的單組件監聽的效果，所以進行了 connect 的組件，它的 parent component 是不知道它得到了哪些數據的，所以就沒法進行整個 DOM Tree 的 Caching。
