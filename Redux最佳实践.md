
## 首先Redux是有自己的三大理念的（Three Principles），所以最佳实践都是基于这三个理念的（有例外情况，后面会介绍）：

－ Single source of truth：应用程序的所有 state 应该被保存在单个 store 中。

－ State is read-only: state 不能被直接修改，只能通过触发 action 来修改。

－ Changes are made with pure functions: 使用纯函数式的 reducer 来处理数据修改逻辑。纯函数意味着没有side effect，也就是处理逻辑不能依赖于全局变量，只能依赖于传入的参数；不能修改传入的参数，返回的应该是一个全新的对象。

## 下面是一些基于这三个理念衍生出的实践：

### 关于 Store

1. API 返回的数据与 view 需要的数据分开放在不同的节点上。API 的数据其实是复用程度更高的数据，与 view 的数据隔离开，一方面可以帮助我们反省 view 中是否有冗余的数据，另一方面可以更容易的读取 API 返回的数据。

2. 存储 API 返回的数据时，应该与最原始的数据保持一致。这样才不会限制我们未来做转换的可能性。

3. View 中只存储不可以被计算出的数据。存储 View 数据时，一定要时刻反省这个到底是可以从其他数据中计算出来的，还是需要缓存的。对于可以从其他数据中计算出来的，就不要重复存储一份了，直接在 selector 中计算出来就可以了。因为冗余数据的同步维护，是非常费力并且容易出错的。

4. 临时数据不一定要进 store。例如查询结果这种数据，可以直接放在 react view 的 state 中。但如果查询结果需要被缓存，切换页面回来后还能看到，那还是要进 store。

### 关于 Action

1. 使用 flux standard action（GitHub - acdlite/flux-standard-action: A human-friendly standard for Flux action objects.）标准化的 action 数据结构可以让维护更容易，也让第三方库的引入更容易。

2. Action 只做简单的参数传递和配置，最多做一些数据格式的转换，例如：把 date 转成 formatted string。尽量把数据处理的逻辑放在reducer中。因为按照语意，action 就只是一个 identifier，reducer 才是负责逻辑处理的。

3. 对于异步请求的 action，action 不要真的 call ajax，只 create ajax options，call ajax 放在 middleware 中。因为 ajax action 往往会触发后续的一些 error action，只有在 middleware 中才能控制 action flow，触发这些后续的 error action。

### 关于 Reducer

1. Reducer 中只处理和保存需要被缓存，也就是无法被 selector 直接计算出的数据，其他的数据逻辑都放在 selector 中计算出来。这个其实和 store 的第三条类似，主要是为了防止冗余数据的出现，所以能用计算得出的数据就不要存储了。

2. 使用 normalizr（GitHub - paularmstrong/normalizr: Normalizes nested JSON according to a schema）处理API数据。这个也是为了防止冗余数据的出现。

3. 针对 API，不同的 domain 应该有不同的 reducer。针对 view，不同的模块应该有不同的 reducer，最后通过 combineReducers 来 combine 到一起。这样做主要是为了有更清晰的模块化。

### 关于 Selector

我们倾向于把所有的 view 的数据逻辑都放在 selector 中，而不是分散到每个 view 的 render 方法中。这样的好处是你很容易在多个 selector 中发现重复的逻辑，抽出共用的方法。但是也有坏处，就是 selector 往往会非常大（我们的 selector function 会单独放一个文件），container 的 props 会非常多，并且 container 需要知道所有子 view 的显示逻辑。不过为了增加代码的复用性，减少重复代码，这些都是值得的。
