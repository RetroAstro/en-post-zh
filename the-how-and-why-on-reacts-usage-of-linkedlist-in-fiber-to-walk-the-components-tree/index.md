> 原文地址：https://medium.com/react-in-depth/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-67f1014d0eb7
>
> 原文作者：[Max Koretskyi aka Wizard](https://github.com/maximusk)

在 React 中，变化检测机制通常被称作协调或者渲染，而 Fiber 正是该机制的最新实现。得益于 Fiber 的底层架构，它让 React 有能力实现许多有趣的特性，例如执行非阻塞的渲染、根据优先级进行组件的更新以及在后台预渲染内容。这些特性在[并发 React 设计哲学](https://twitter.com/acdlite/status/1056612147432574976)中被称作 **time-slicing** 。

对于开发者来说，除了解决应用中的实际问题，**从工程角度来看，应用本身的内部实现同样具有广泛的吸引力。并且在应用的源码里也有着足够丰富的知识能够帮助我们成为真正的开发者。** 

如果你在 Google 上搜索 “React Fiber” ，你会发现很多与之相关的文章。但是除了 [Andrew Clark 的笔记](https://github.com/acdlite/react-fiber-architecture) 以外，其他都是十分高深的解释。在本篇文章中，我会引用该笔记中的某些内容并**对 Fiber 中一些特别重要的概念进行详细的阐述。** 在你读完整篇文章之后，你将会有足够的知识去理解 Lin Clark 在 [2017 年的 ReactConf](https://www.youtube.com/watch?v=ZCuYPiUIONs) 上所讲解的 work loop 模型。当然如果你能够花费时间去阅读源码，那对 Fiber 的理解就会更加透彻。

## 背景

Fiber 架构主要分为两个阶段：协调 / 渲染阶段与提交阶段。在 React 源码中协调阶段通常被称作“渲染阶段” 。在这个阶段中 React 会遍历整个组件树然后执行以下的任务：

* 更新 state 和 props
* 调用生命周期钩子函数
* 检索当前组件树中的子元素
* 与先前组件树中的子元素进行比较
* 找出需要执行的 DOM 更新

**所有的这些活动在 Fiber 内部都被称作 work 。** 那些需要被完成的 work 类型取决于 React 元素的类型。例如，对于 `Class Component` 来说 React 需要实例化一个 class ，而 `Functional Component` 则不需要。如果你对它们感兴趣，[这里](https://github.com/facebook/react/blob/340bfd9393e8173adca5380e6587e1ea1a23cefa/packages/shared/ReactWorkTags.js#L29-L28)有 Fiber 中的所有 work 类型。这些活动正是 Andrew 提及过的：

> 在处理 UI 问题时，如果一次**执行太多的工作**，将会导致屏幕上的动画掉帧。

那么该如何理解这里的“一次执行所有工作”呢。好吧，基本上，如果 React 同步地去遍历整个组件树然后再依次执行相应组件的 work ，应用中逻辑代码的执行时间可能就会超过有效的 16 ms 。而这正是造成掉帧和视图不一致的罪魁祸首。

那么有解决的办法吗？

> 在新的浏览器（以及 React Native）中实现的 API 正好可以帮助我们解决这个问题。

这个新的 API 叫做 [requestIdleCallback](https://developers.google.com/web/updates/2015/08/using-requestidlecallback) ，它能够生成一个函数队列，并在浏览器空闲的时候依次执行这个队列中的函数。下面是它的用法：

```js
requestIdleCallback((deadline)=>{
    console.log(deadline.timeRemaining(), deadline.didTimeout)
})
```

如果我把上面的代码放到浏览器的控制台中执行，Chrome 会输出 `49.9 false` 。这基本上代表着我还有 `49.9ms` 来完成我需要做的任何工作，并且我没有用完所有分配的时间，直到 `deadline.didTimeout` 变为 `true` 。需要注意的是 `timeRemaining` 会随着浏览器处理工作的频率而改变，因此需要不断地被检查。

> 事实上 **`requestIdleCallback`** 函数在有些时候会显得过于苛刻，并且经常会因为执行次数不够而导致 UI 渲染的不流畅，为此 React 团队不得不重新[实现他们自己的版本](https://github.com/facebook/react/blob/eeb817785c771362416fd87ea7d2a1a32dde9842/packages/scheduler/src/Scheduler.js#L212-L222)。

如果我们把 React 在组件中执行的所有活动放到 `performWork` 中去，然后使用 `requestIdleCallback` 函数去调度相应的 work ，我们的代码可能会像下面这样：

```js
requestIdleCallback((deadline) => {
    // 只要有足够的时间，就执行组件树中一部分的 work
    while ((deadline.timeRemaining() > 0 || deadline.didTimeout) && nextComponent) {
        nextComponent = performWork(nextComponent);
    }
})
```

