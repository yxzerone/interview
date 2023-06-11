# React

## React 事件机制

React 中的事件并没有绑定到对应组件的真实 DOM 上，而是采取**事件委托**的方式，将所有的事件绑定到 `document`，进行统一处理。这种方式减少了内存消耗，还能在组件挂载或销毁时统一订阅和移除事件。

但需要注意的是，事件的执行顺序是，原生对象先执行，合成对象后执行。如果原生事件阻止了事件冒泡，可能导致合成事件不执行，因为合成事件必须冒泡到 document 上才能执行，所以应该**尽量避免混用原生事件和合成事件**。

此外，React 中的事件对象也不是浏览器的原生对象，而是由 React 包装的合成事件（`SyntheticEvent`）。因此，在 react 中，应该**调用 `event.preventDefault` 阻止事件冒泡**，调用 `event.stopPropagation` 方法是无效的。

实现合成事件的好处：

- 合成事件是一个跨浏览器的原生事件对象包装器，它可以消除浏览器之间事件处理的差异，提高 web 应用的兼容性。

- react 中有一个事件池，用于管理合成对象的创建和销毁。当事件触发时，就会从池子中复用对象，以减少对原生对象的创建和销毁。

## React 高阶组件（HOC）、Render props、hooks

HOC 是一种基于 React 的组合特性形成的一种设计模式，用于复用组件逻辑。具体来说，**高阶组件是纯函数，接受一个组件和额外的参数（可选），并返回一个新的组件。**

- 优点：逻辑通用，可复用，不影响包裹组件的内部逻辑。

- 缺点：多个高阶组件如果存在相同名称的 props，则存在命名覆盖问题。

适用场景：

1. 权限管理：可以利用条件渲染决定是否显示组件，即对组件进行权限控制。

2. 组件渲染性能追踪：继承类组件覆盖生命周期钩子，可以方便记录组件的渲染时间。

3. 页面复用。

"render prop"是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术。具体来说，**组件接受一个返回 React 元素的函数（可以将组件的 state 作为参数），从而将函数的渲染逻辑注入到组件内部。**

- 优点：不会产生无用的空组件加深层级，不用担心命令问题。

- 缺点：无法在 render 函数访问数据。如果有多个这样的组件嵌套，容易造成嵌套地狱。

Hooks 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。通过自定义 hook，可以复用代码逻辑。

需要注意的是：hooks 只能在组件顶层使用，不可在分支语句中使用。

- 优点：使用直观，不存在命名、嵌套问题。

- 缺点：只能在函数组件中使用。

## React-Fiber

React v15 在渲染时，会递归对比 vDOM 树，找出需要重新渲染的节点，然后同步更新。在这个过程中，React 会一直占据浏览器资源，如果耗时过长，会让用户感觉卡顿。React-Fiber 通过协程，将这一过程变为可中断，适时让出浏览器资源。

这样做有以下几个好处：

- 分批延时执行 DOM 操作，避免一次性操作大量 DOM。

- 合理分配浏览器资源，提高浏览器对用户的响应速率。

## React.PureComponent

在 React 中，当 props 或者 state 发生变化，组件就会触发 render 函数。为了减少更新页面（不必要的 render 执行次数），可以在生命周期函数 `shouldComponentUpdate` 中返回 `false`。

`PureComponent` 利用的就是这个原理：当组件更新时，如果组件的 props 或者 state 都没有改变，render 函数就不会触发。这就省去了比较 vDOM 的时间，从而提升了性能。

不过，`pureComponent` 中的 `shouldComponentUpdate`  进行的是**浅比较**。也就是说，只要 props 或者 state 的指向未发生变化（修改属性值不会修改对象的指向），render 函数就不会触发。

## Component, Element, Instance

- 元素（Element）: react 中的元素是一个不可变的普通对象，用于描述一个真实 DOM 节点或者 React 组件。

- 组价（Component）: React 中的组件有两种形式：带有 render 方法的类或者一个函数。这两种情况下，它都将 props 作为输入，把返回的一棵元素树（element tree）作为输出。

- 实例（Instance）: 类组件中的 `this`，用来存储 state 或者声明周期函数。函数组件没有实例。

## 有状态组件和无状态组件

### 有状态组件

有状态组件基于外部传入的 props 和自身的 state（状态）进行渲染。一般情况下指类组件（当类组件不需要管理状态时，也可充当无状态组件），但在新版 React hooks 加持下，函数组件也可以实现状态。

使用场景：组件内部需要依赖自身数据进行操作时。

优点：在类组件下，可以使用生命周期，对组件进行更精细的操作。

缺点：有状态组件如果使用较多，容易触发生命周期钩子，影响性能。

### 无状态组件

不依赖自身状态 state，而是依赖于外部传入的 props，这样的组件称为无状态组件。当不需要使用生命周期钩子时，应当首选无状态函数组件。

优点：

- 代码简洁。渲染只取决于输入（props），无副作用。

- 组件不需要实例化，无生命周期。相对于有状态组件，性能较高。

缺点：无法控制组件的重渲染，当父组件更新时，无论子组件的 props 有没有更改，子组件依然会重渲染，但新版 react 解决了这个问题。

**memo 函数，通过记忆组件渲染结果的方式复用组件。这意味着，如果 props 没有发生改变，React 将跳过组件渲染并直接复用最近一次的渲染结果。**

**React.memo 仅检查 props 变更**。如当 state 或 context 发生变化时，它仍会重新渲染。

React.memo 默认情况下只会对复杂对象做**浅层对比**，如果需要控制对比过程，则需要自定义比较函数作为第二个参数。

一般情况下，不需要对组件使用 memo 函数，因为 props 的比较也需要花销。**但如果组件的渲染耗时较长，则可以使用，这样可以显著提升渲染效率。**

## Fragment

在 React 中，组件渲染函数返回的元素必须且只能有一个根元素。为了避免添加多余的 DOM 节点，可以使用 `Fragment` 标签。`Fragment` 标签不会渲染出任何标签。

## Ref（组件如何获取组件的真实 DOM 元素）

可以使用 `ref` 获取某个子节点的 DOM 元素或者 React 组件实例对象，但必须在组件挂载后才能使用，因为渲染阶段真实 DOM 还没有生成。

注意：

- **函数组件没有实例，因此在函数组件上，绑定属性 `ref` 是无效的。但如果确实需要，可以使用 useImperativeHandle 和 forwardRef 自定义函数组件暴露的方法和数据。**

- **不应该过度使用 ref。**

有三种实现方式：

- **字符串格式**：**这种方式只适用于类组件。**`ref prop` 可以接受一个字符串，该字符串代表要使用的 ref 节点名称。在组件挂载后，通过 `this.refs.xxx` 访问节点实例。

- **函数格式**：`ref prop` 可以接受一个函数，参数函数有一个参数，该参数对应节点实例。

- **createRef / useRef 方法**：`ref prop` 可以接受一个 create Ref 或 useRef 方法创建的一个 ref 实例（函数返回值），实例的 current 属性代表节点实例。两个方法使用场景不同，前者用于类组件，后者属于 hook，只能用于函数组件。

使用场景：

- 处理焦点、文本选择或者媒体的控制。

- 触发动画或者过渡。

- 集成第三方 DOM 库。

## Portals（插槽）

在 WEB 应用中，有些元素需要挂载到更高层级的位置，最典型的场景就是一些弹出框。

React 16 提供了一种方案 `ReactDOM.createPortal`，它可以分离子组件和父组件，让子组件挂载到 DOM 树的任何地方。

这个函数接受两个参数，第一个参数是可渲染的 React 元素。第二个参数是一个 DOM 元素，表示将要挂载的容器元素。

## 避免不必要的 Render

React 基于虚拟 DOM 和高效 DIFF 算法，实现了对 DOM 的最小颗粒的更新。在一般情况下，React 的渲染效率足以胜任。但在个别复杂业务场景下，性能问题依然存在。此时就需要采取一些措施提升性能，避免不必要的渲染。

- **shouldComponentUpdate / PureComponent**：在类组件中，可以通过覆盖 `shouldComponentUpdate` 或者继承 `PureComponent` 来阻止父组件更新导致的子组件更新。

- **利用高阶组件**：在函数组件中，没有 `shouldComponentUpdate` 钩子，但是可以利用高阶组价封装一个类似于 `PureComponent` 的功能。

- **React.memo**：这个函数的 React 16 的一个新 API，用于缓存组件最新一次的渲染结果。其本质上也是一个高阶组件，但是只能用于函数组件。

## React-Intl

React-Intl 是雅虎的语言国际化开源项目 FormatJS 的一部分，它提供了一系列的 React 组件，包括数字格式化、字符串格式化、日期格式化等。

在 React-Intl 中，可以配置不同的语言包，他的工作原理就是根据需要，在语言包之间进行切换。

## React context

在 React 中，数据传输一般使用 props，以维持单向数据流，这样可以使父子组件之间的关系变得简单可预测。但是，如果组件之间层层嵌套，props 就需要层层传递，这样显然太繁琐。

Context 提供了一种在组件树中共享数据的方式，支持跨组件式的数据传递。从某种角度来说，context 就是组件树内共享的 store。

就组件的数据通信来说，并不推荐优先使用 context。如果 props 和状态管理都不是最佳方案时，再考虑使用 context，但前提是能做到高聚合，不破坏组件树之间的依赖关系。

## 受控组件和非受控组件

### 受控组件

在需要收集用户输入时，为组件的每一个表单元素绑定一个 `change` 事件， `change` 事件用于更新组件的 state。这样的组件在 React 中被称为**受控组件**。React 官方推荐使用受控表单组件。

触发流程：用户输入发生变化 -> 触发 change 事件 -> 更新组件的 state -> 重新渲染组件。

优点：表单元素的值全由 React 管理（组件的 state 与表单元素的 value 或 checked 属性相对应），使得组件的整个状态可控。

缺点：当组件内的表单元素很多时，就需要为每一个表单元素绑定 change 事件，这会让代码变得很臃肿。

### 非受控组件

如果 React 组件没有接管表单元素（组件 state 没有表单元素对饮的属性），则可以被称为非受控组件。

优点：相比于受控组件，非受控组件不需要为每一个表单元素绑定事件，代码量更少。

缺点：因为用户输入的数据都存储在真实 DOM 节点中，所以在非受控组件中，需要使用 ref 从表单元素获取值。

## React.forwardRef

`React.forwardRef` 会创建一个 React 组件，这个组件能够将 `ref prop`（组件接受的 ref 属性）转发到子组件或者其渲染树下的 DOM 元素。

使用场景：

- 父组件需要获取隔代组件或者子组件内部的 DOM 元素时。

- 函数组件需要暴露数据或方法时。

## 类组件与函数组件的异同

相同点：

- 组件是 React 可复用的最小代码片段，它们会返回要在页面中渲染的 React 元素。

- 在使用方法和最终渲染结果上都是完全一致的。在非极端情况下，两类组件的性能也可以说是完全一致的。

不同点：

- 类组件基于面对对象编程，主打继承、生命周期等核心概念。函数组件基于函数式编程，主打 immutable、没有副作用、引用透明等特点。

- 相较于类组件，函数组件更加轻量化。

- 性能优化上，类组件的优化主要依靠 shouldComponentUpdate 隔断渲染，而函数组件则基于 React.memo 复用渲染结果提升性能。

- React hooks 是 React 最新推出的方案之一，从未来趋势看，函数组件称为未来主推的方案。

## state 和 props

`props`: 组件外部传入的参数，是组件通信的一种方式。只读，不可变，以保证相同的 props 渲染出的是一样的内容。只能在组件外部通过生成新的 props 来替换，从而重新渲染组件。

`state`: 组件自身的状态，只能在组件内部初始化，相当于组件的私有属性。在组件外部无法进行访问和修改，组件内部通过 `setState` 更新自身的状态，从而重新渲染组件。

## props 验证

PropTypes 用来验证传入的 prop 是否有效，当 prop 传入的类型和验证类型不符时，就会在控制台发出警告信息。它可以有效避免 React 应用在编写时的意外情况，使得程序更易读。

当然，如果 React 集成了 TypeScript，则可以使用 TypeScript 定义接口限制 prop 的传递类型。

## props 改变时调用的钩子

### componentWillReceiveProps（已废弃）

这个钩子接受新的 props 作为参数，并在 render 函数执行前调用（在组件挂载期间不会调用）。

如果组件 state 的部分数据依赖于 props，则可以在这个钩子里及时更新 state。

同样的，可以将依赖于 props 的数据请求放在这个钩子里执行，以减少父组件中的请求。

### getDerivedStateFromProps（16.3 引入）

`getDerivedStateFromProps` 是一个**静态函数**，功能类似于 `componentWillReceiveProps`。但不同的是，无论是在组件挂载期间，还是在组件更新时，都会触发这个钩子函数。

钩子函数接受**新 props 和 state** 作为参数，返回值用于更新 `state`，但如果不更新 `state`，则需要返回 `null`。

## setState

`setState` 用于修改组件的状态，并通知 React 重新渲染该组件及其子组件，是更新用户界面的主要方式。

它接受两个参数，第一个参数是合并到 state 的对象或者一个更新函数，第二个参数是组件更新后执行的回调函数。

调用 `setState` 时，组件的 `state` 并不会立刻发生改变。出于对性能的考虑，`setState` 只是将修改的 `state` 加入一个队列，然后在合适的时机将队列中的状态修改合并成一次，最终只产生一次组件状态更新，这在大型应用程序中对性能的提升至关重要。

需要注意的是，只要一轮事件循环没有结束，压入队列这个操作就不会停止。如果在一轮事件循环中，多次对 `state` 中的同一个属性进行修改，则 React 只保留最后一次修改。

调用 `setState` 函数之后，React 会将传入的参数对象与组件当前的状态合并，然后触发调和过程。经过调和过程，React 会以相对高效的方式根据新的状态构建 React 元素树并且着手重新渲染整个UI界面。

在 React 得到元素树之后，React 会自动计算出新的树与老树的节点差异，然后根据差异对界面进行最小化重渲染。在差异计算算法中，React 能够相对精确地知道哪些位置发生了改变以及应该如何改变，这就保证了按需更新，而不是全部重新渲染。

如果在短时间内频繁 `setState`。React会将 `state` 的修改压入队列中，并在合适的时机，批量更新 `state` 和视图，从而提高性能。

## setState 是同步的还是异步的？

如果将 `setState` 设计成同步，则当 `state` 一旦发生变化时，组件都会发生一次更新。这种情况下，`render` 函数会被频繁触发，界面频繁渲染。这很显然，效率是很低的。但如果只更新 `state`，不更新界面，父组件的 `state` 和子组件的 `props` 就无法保持一致。因此，将 `setState` 设计成异步，可以显著地提升性能，降低渲染次数。

实际上，`setState` 并不单纯是同步的或是异步的，它的表现会因调用场景的不同而不同。

- **异步**：React 生命周期和合成事件等 React 可以控制的地方。这些场景走合并操作，延迟组件的更新。

- **同步**：原生事件、setTimeout 和 setInterval 等 React 无法控制的地方。这些场景下，只能同步更新。

## replaceState

`replaceState` 方法功能与 `setState` 类似，不同的是，replaceState 是**完全替换**组件的 `state`，相当于重新赋值，而 setState 是合并，相当于 `Object.assgin`。