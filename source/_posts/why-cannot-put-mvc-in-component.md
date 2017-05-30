title: 我们为什么不在组件中使用 mvc
date: 2015-12-22 20:01:22
tags:
---

1. 首先，我们使用 `mvc`，目标就是把相应的处理划分到相应的 `m` / `v` / `c` 中去。所以，一个 Component 中也有一个的 `mvc`，说不过去。
2. 其次，Component 是多实例的，而多实例之间多存在数据关联关系。简单点来讲，也就是说 Component 实例需要同一个数据源，来除去数据加载冗余和管理共享数据更新问题。这两个问题并不是那么容易搞定。即使搞定了，我们也会困惑，为什么一个 <Select> 组件还需要依赖一个 SelectDataSource 模块。如果只有一个 `m`，那这个事情就变得明确的很多。我们必须向 <Select> 提供 datasource 属性，否则我就罢工了。
3. 最后，如果一个 `mvc` 整体作为 Component 输出，那么必须保证这个 Component 能够提供一个组件应有的支持。比如，在 React 中，Component 生命周期相关的两个重要接口 `componentDidMount` 和 `componentWillReceiveProps`。 `componentDidMount` 实际上还是比较容易处理的，而 `componentWillReceiveProps` 就麻烦了。`mvc` 是有状态的，在接受了 `nextProps` 之后，action 如何处理呢？由于我们 `m` 是有私有状态的，不一定能从 `props` 得到所有的 `m` 状态。所以，首先需要 diff 一下 currentProps 和 nextProps，再根据 diff 来更新 `m`，最后交给 `v` 更新。在 `redux` 框架下，可能会 dispatch 一个这样的东西给 reducer：

    ```
    {
        type: 'repaint',
        payload: nextProps
    }
    ```

reducer 看到这个 action 的时候估计就要哭了吧。
