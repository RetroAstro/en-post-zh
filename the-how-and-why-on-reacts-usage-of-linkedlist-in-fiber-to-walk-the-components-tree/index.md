> 原文地址：https://medium.com/react-in-depth/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-67f1014d0eb7
>
> 原文作者：[Max Koretskyi aka Wizard](https://github.com/maximusk)

在 React 中，变化检测机制通常被称作协调或者渲染，而 Fiber 正是该机制的最新实现。得益于 Fiber 的底层架构，它让 React 有能力实现许多有趣的特性，例如执行非阻塞的渲染、根据优先级进行组件的更新以及在后台预渲染内容。这些特性在[并发 React 设计哲学](https://twitter.com/acdlite/status/1056612147432574976)中被称作 **time-slicing** 。

对于开发者来说，除了解决应用中的实际问题，**从工程角度来看，应用本身的内部实现同样具有广泛的吸引力。并且在应用的源码里也有着足够丰富的知识能够帮助我们成为真正的开发者。** 

如果你在 Google 上搜索 “React Fiber” ，你会发现很多与之相关的文章。但是除了 [Andrew Clark 的笔记](https://github.com/acdlite/react-fiber-architecture) 以外，其他都是十分高深的解释。在本篇文章中，我会引用该笔记中的某些内容并**对 Fiber 中一些特别重要的概念进行详细的阐述。** 在你读完整篇文章之后，你将会有足够的知识去理解 Lin Clark 在 [2017 年的 ReactConf](https://www.youtube.com/watch?v=ZCuYPiUIONs) 上所讲解的 work loop 模型。当然如果你能够花费时间去阅读源码，那对 Fiber 的理解就会更加透彻。

## 背景

