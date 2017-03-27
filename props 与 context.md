##  props 与context 的适用场景

React的 context 和全局变量非常相似，在大多数场景下，我们都应该尽量避免使用。适合使用 context 的场景包括传递登录信息、当前语言以及主题信息等。
另外react-redux的Provider 组件就使用了context 来传递store.
