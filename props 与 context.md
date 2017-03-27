##  props 与context 的适用场景

> 严格来说,state 应该被称为内部状态（local state). 内部状态的操作配合React事件系统 可以实现 一些用户交互的功能

> React的 context 和全局变量非常相似，在大多数场景下，我们都应该尽量避免使用。适合使用 context 的场景包括传递登录信息、当前语言以及主题信息等。
另外react-redux的Provider 组件就使用了context 来传递store.

> Props 和 context 则用于在组件间传递数据。使用props 传递数据简单清晰， 但是跨级传递非常麻烦。使用 context 可以实现跨级传递数据 ，但是会降低组件的复用性，因为这些组件依赖“上下文” 。所以尽量只使用context 传递登录信息、当前语言以及主题信息等全局数据 。
