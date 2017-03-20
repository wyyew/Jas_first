# React redux 学习笔记

@(react)[react|redux|react-redux|es6|函数式编程|react-router]

## 异步Action

一般情况下，每个 API 请求都需要 dispatch 至少三种 action：

- **一种通知 reducer 请求开始的 action。**

> 对于这种 action，reducer 可能会切换一下 state 中的 isFetching 标记。以此来告诉 UI 来显示加载界面。

- **一种通知 reducer 请求成功的 action。**

> 对于这种 action，reducer 可能会把接收到的新数据合并到 state 中，并重置 isFetching。UI 则会隐藏加载界面，并显示接收到的数据。

- **一种通知 reducer 请求失败的 action。**

> 对于这种 action，reducer 可能会重置 isFetching。另外，有些 reducer 会保存这些失败信息，并在 UI 里显示出来。
