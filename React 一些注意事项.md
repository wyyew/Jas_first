##  props 与context 的适用场景

> 严格来说,state 应该被称为内部状态（local state). 内部状态的操作配合React事件系统 可以实现 一些用户交互的功能

> React的 context 和全局变量非常相似，在大多数场景下，我们都应该尽量避免使用。适合使用 context 的场景包括传递登录信息、当前语言以及主题信息等。
另外react-redux的Provider 组件就使用了context 来传递store.

> Props 和 context 则用于在组件间传递数据。使用props 传递数据简单清晰， 但是跨级传递非常麻烦。使用 context 可以实现跨级传递数据 ，但是会降低组件的复用性，因为这些组件依赖“上下文” 。所以尽量只使用context 传递登录信息、当前语言以及主题信息等全局数据 。

> 无状态函数是没有实例化对象的，因些无法使用生命周期函数 ，也没有内部状态。所以如果 你的组件 需要使用生命周期函数或者内部状态，请使用类编写该组件。

## 组件，ReactElement与组件 实例的区别

const componnetInstance = render(<App />, document.querySelector("#root");

1. console.log(App); // function App() {} --组件
2. console.log(<App />); // 一个$$typeof 为 Symbol(react.element)的对象 -- ReactElement
3. console.log(componentInstance); //一人名为App的对象 --- 组件实例 

## 事实上，Redex中的异步操作就两件事

- 使用Thunk 中间件使 action 创建函数可以进行异步操作。
- 在进行异步请求的前后 各发起一次 action 去改变state ，进而改变视图和执行一些其他的逻辑
