# Vue 生命周期

Vue 组件有一套完整的执行流程：开始创建、初始化数据、模板编译 ->  -> 挂载 DOM -> 渲染、更新 -> 卸载，这被称为是 Vue 组件的生命周期。

生命周期可以分为 3 个阶段：

- 挂载阶段：组件第一次挂载到 DOM 树，在整个生命周期中只执行一次。

- 更新阶段：重新渲染更新组件。通常是因为组件依赖的响应式数据发生了变化。

- 卸载阶段：卸载 Vue 组件实例，在整个生命周期中只执行一次。

## Vue 生命周期钩子

1. `beforeCreate`（在组件实例创建后，`data`、`computed` 等选项处理前调用）: 此时响应式数据、计算属性、数据监听及自定义方法未设置，也就是说，在这个钩子里，组件实例无法访问由 `data`、`computed` 和 `methods` 等选项定义的属性和方法。

2. `created`（处理选项完成后调用）：传入组件的选项配置完成，可以访问选项定义的属性和方法，但是此时组件挂载阶段还未开始，因此 `$el` 属性仍不可用。这个钩子通常用于初始化某些属性值。

3. `beforeMount`（组件挂载前调用）：此时，组件即将执行首次 DOM 渲染，但还未创建 DOM 节点。

4. `mounted`（组件挂载后调用）：所有同步子组件挂载完成，可以执行与 DOM 相关的操作，比如访问属性 `$el`，发起请求。这个钩子通常用来执行与组件 DOM 树相关的副作用。

5. `beforeUpdate`（组件 DOM 树更新前调用，通常是因为响应式数据发生了变化）：此时，虽然响应式数据更新了，但是组件的 DOM 树还没有更新。因此，在这个钩子里，可以在 DOM 更新前访问 DOM 状态，也可以继续更改响应式数据。

6. `updated`（组件 DOM 树更新后调用）：此时，组件 DOM 树已经更新，可以执行与 DOM相关的操作。**但需要注意的是，不能在这个钩子里更改响应式数据，否则可能会导致无限的更新循环。**如果需要在某个特定的响应式数据更改后访问更新后的 DOM，可以使用 `nextTick`。

7. `beforeDestroy (Vue2) / beforeUnmount (Vue3)`（组件卸载前调用）：此时，组件实例仍保留全部功能。

8. `destroyed (Vue2) / unmounted (Vue3)`（组件卸载后调用）：此时，所有的子组件都已经被卸载，所有的响应式作用都停止运行。在这个钩子里，可以手动清理一些副作用，比如：定时器、DOM 事件监听器或者与服务器的连接等。

## 异步数据请求

因为在钩子函数 `created`、`beforeMount` 和 `mounted `中，`$data` 及响应式数据已经处理完成，所以可以在这三个钩子里发送请求，然后将服务器返回的数据赋值给响应式数据。

但更推荐在 `created` 中发送异步请求。

1. 能更快接受到服务器数据，减少页面加载时间，用户体验更好。

2. SSR 不支持 `beforeMount`、`mounted` 钩子函数，请求放在 `created` 中更一致。

## 子组件和父组件生命周期钩子的执行顺序

挂载阶段：

1. 父组件：`beforeCreate` -> `created` -> `beforeMount`
2. 子组件：`beforeCreate` -> `created` -> `beforeMount` -> `mounted`
3. 父组件：`mounted`

更新阶段：

1. 父组件：`beforeUpdate`
2. 子组件：`beforeUpdate` -> `updated`
3. 父组件：`updated`

卸载阶段（vue2）：

1. 父组件：`beforeDestroy`
2. 子组件：`beforeDestroy` -> `destroyed`
3. 父组件：`destroyed`

卸载阶段（vue3）：

1. 父组件：`beforeUnmount`
2. 子组件：`beforeUnmount` -> `unmounted`
3. 父组件：`unmounted`

## setup

Vue3 中增加了 `setup` 钩子。它是使用组合式 API 的入口，相当于 `beforeCreate` 和 `created` 两个钩子，但 `setup` 的执行时机却早于这两个钩子。

## keep-alive

`keep-alive` 是 Vue 提供的一个内置组件，用来缓存组件。如果为一个组件包裹了 `keep-alive`，那么它会多出两个生命周期选项：`deactivated` 和 `activated`，这两个钩子适用于缓存组件及其所有子孙组件。

- `deactivated`：缓存的组件**卸载时**或者**从 DOM 移除时**调用。

- `activated`：缓存的组件**挂载时**或者**插入 DOM 时**调用。

默认情况下，一个组件被替换后会被销毁。但当包裹了 `keep-alive` 的组件从 DOM 树移除时，它会被加入缓存，变为**不活跃状态**。同时因为缓存的组件不会被真正销毁，所以卸载生命周期钩子（`beforeUnmount` / `unmounted`）将不再触发。

同样的，当缓存的组件重新插入到 DOM 树时，Vue 会在缓存中寻找这个组件，然后**重新激活**它。
