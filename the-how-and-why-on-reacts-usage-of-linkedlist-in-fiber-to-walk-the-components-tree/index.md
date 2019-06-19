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

我们在一个组件上执行其相应的 work 然后返回下一个需要被处理的组件的引用。同步地处理整个组件树是不合理的，就像 React [先前实现的协调算法](https://reactjs.org/docs/codebase-overview.html#stack-reconciler)一样。Andrew 也提到了这个问题：

> *为了使用这些 API ，你需要一种方法将渲染时的 work 拆分为增量单位。*

为了解决这个问题，React 团队不得不重新实现遍历组件树的算法，将其**从依赖于内置栈的同步递归模型转换为基于链表和指针的异步模型**。而这正是 Andrew 在下面所写到的：

> *如果依赖于内置的调用栈，React 将会持续地工作直到栈被清空…… 如果我们可以随时中断调用栈并且还能手动操作调用帧，那不是很好吗？而这正是 React Fiber 想要达到的目标。**Fiber 是对先前基于栈的协调算法的重新实现，且专门用于 React 组件。** 你可以把单个 fiber 想象成一个虚拟的栈帧。* 

这就是我马上将要解释的内容。

### 关于栈的解释

我假设读者对栈的概念都很熟悉。如果你的代码在断点处停止，那么在浏览器的 debugging 工具上显示的就会是一个类似栈的东西。以下是[维基百科](https://en.wikipedia.org/wiki/Call_stack?fbclid=IwAR06VWEQnwoEawg0NsoR8loBJwIbmPWsXXKqbAuOFBjkawHThK7zlIBsJ_U#Structure)上的一些相关引用和图表：

> *在计算机科学中，**调用栈**是一种栈的数据结构，它储存着有关计算机活动子程序的信息。使用调用栈的主要原因是为了跟踪每个活动子程序在执行完成时应该返还控制的点。**调用栈**由许多个**栈帧**组合而成，每个栈帧对应着一个暂未以 **return** 语句终止的子程序调用。例如，如果一个名为 `DrawLine` 的子程序正在执行，并且它是被叫做 `DrawSquare` 的子程序调用的，那么在调用栈顶部的内容可能就会像下面所展示的图片一样。*

![](./assets/call-stack.png)

### 为什么栈会与 React 相关？

正如我们在本文第一部分所定义的，React 会在协调 / 渲染阶段遍历整个组件树并且会为组件执行相应的工作。先前的协调算法使用的是依赖于内置栈的同步递归模型来遍历整个树。关于协调[官方文档](https://reactjs.org/docs/reconciliation.html#recursing-on-children)讲述了整个过程并且谈论了许多关于递归的内容：

> *默认情况下，当对 DOM 节点的子节点进行递归时，React 会同时迭代子节点的列表，并且在产生差异时生成突变。* 

你可以这样想，每次的递归调用都会向栈中增加一个帧，并且这个过程是同步的。假设我们有以下的组件树：

![](./assets/components-tree.png)

我们用 `render` 函数代表对象。你可以将它们视为组件的实例：

```js
const a1 = {name: 'a1'};
const b1 = {name: 'b1'};
const b2 = {name: 'b2'};
const b3 = {name: 'b3'};
const c1 = {name: 'c1'};
const c2 = {name: 'c2'};
const d1 = {name: 'd1'};
const d2 = {name: 'd2'};

a1.render = () => [b1, b2, b3];
b1.render = () => [];
b2.render = () => [c1];
b3.render = () => [c2];
c1.render = () => [d1, d2];
c2.render = () => [];
d1.render = () => [];
d2.render = () => [];
```

