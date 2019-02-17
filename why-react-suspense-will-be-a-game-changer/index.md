> 原文地址：https://medium.com/react-in-depth/why-react-suspense-will-be-a-game-changer-37b40fea71ec
>
> 原文作者：[Julian Burr](https://github.com/julianburr)

在本篇文章中，我不想太深入解释有关 React Suspense 的实现细节和它内部的工作原理，因为已经有[很多](https://auth0.com/blog/time-slice-suspense-react16/)[优秀的](https://hackernoon.com/building-a-polyfill-for-react-suspense-f1c7baf18ca1)[博客文章](https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html)、[视频](https://www.youtube.com/watch?v=SCQgE4mTnjU)和[讨论](https://www.youtube.com/watch?v=z-6JC0_cOns)做过这些事情了。相反，我更愿意把重点放在 Suspense 将会如何影响在应用开发时我们对加载状态和架构应用的思考。

## Suspense 简要介绍

鉴于有些人可能没有听说过 Suspense 或者根本不了解它，因此我会先给出一个关于 Suspense 的简要总结。

在去年冰岛举行的 JSConf 大会上，Dan Abramov 介绍了 Suspense 。在解决 React 应用中有关异步数据获取的难题时，Suspense 被称作是对开发者体验极大改进的 API 。这是件很令人兴奋的事情，因为每个正在构建动态 web 应用的开发者都知道这是一个主要痛点，并且也是带来巨大模版代码的原因之一。

Suspense 同样会改变我们对加载状态的思考方式，它不应该与加载组件或者数据源耦合，而应以 UI 关注点存在。我们的应用应该在用户体验中显示有意义的 spinner 。Suspense 通过解耦以下关注点帮助我们实现这一点：

**Suspense 并不关心你暂停（多少次）的原因，因此一个简单的 spinner 就能和代码分割、数据加载、图片加载等场景组合在一起。无论下面的树需要什么。如果网络速度足够快，甚至可以不显示 spinner 。**

Suspense 不仅在数据加载时有用，它甚至可以被应用到任何异步数据流中去。例如代码分割或图片加载。`React.lazy` 结合 `Suspense` 组件在 React 最新稳定版本中已经可以使用，这允许我们进行动态导入的代码分割，而无需手动处理加载状态。包含数据加载功能在内的完整 Suspense API 将会在今年之内发布，不过你可以通过 alpha 版本提前使用它。

Suspense 的设计思想是让组件具有“暂停”渲染的能力，例如，组件需要从外部资源中加载额外数据时。等到数据都加载完成，React 才会尝试重新渲染组件。

React 使用 Promise 来实现该功能。组件能在 render 方法调用时抛出 Promise（或者任何在组件渲染时调用的方法，例如，新的静态方法 `getDerivedStateFromProps` ）。React 会捕获 Promise 并沿着组件树往上寻找最近的 `Suspense` 组件，并且它会充当一种边界。`Suspense` 组件接收一个名为 `fallback` 的 prop，只要它的子树中有任何组件处于暂停状态，`fallback` 中的组件就会立即被渲染。

React 还会跟踪抛出的 Promise 。组件中的 Promise 一旦 resolve ，React 就会尝试去继续渲染该组件。因为我们假定由于 Promise 已经被 resolve ，这也就意味着暂停的组件已经具有正确渲染所需的全部数据。为此，我们使用某种形式的缓存来存储数据。该缓存取决于每次渲染时数据是否可用（如果可用就会像从变量中取值一样读取它），若数据没有准备好，则会触发 fetch 然后抛出 Promise 以便 React 捕获。如上所述，这并不是数据加载所独有的，任何可以使用 Promise 来描述的异步操作都可以充分利用 Suspense ，显然代码分割是一个非常明显且流行的例子。

Suspense 的核心概念与[错误边界](https://reactjs.org/docs/error-boundaries.html)非常相似。而错误边界在 React 16 中被介绍为能够在应用的任何地方捕捉未捕获的异常，同样的它通过在树中放置组件（在这种情况下为任何带有 componentDidCatch 生命周期方法的组件）来处理从该组件下面抛出的所有异常。无独有偶，`Suspense` 组件捕获任何由子组件抛出的 Promise ，不同的是我们并不需要一个特定的组件来充当边界，因为 `Suspense` 组件自己就是，它可以让我们定义 `fallback` 来决定后备的渲染组件。

![1](./assets/1.gif)

这样的功能显著地简化了我们对应用中加载状态的思考方式并且让我们作为开发者的心智模型与 UX 和 UI 设计师更加一致。

设计师通常并不会考虑数据源，而是更多的考虑用户界面或应用程序的逻辑组织和信息层次结构。你知道还有谁不在意数据源吗？答案是用户。没有人会喜欢成千上万个加载时的 spinner ，并且其中的一些只会闪烁几毫秒，当数据加载完成时，页面内容将会上下跳动。

## 为什么 Suspense 被称作是巨大的突破呢？

