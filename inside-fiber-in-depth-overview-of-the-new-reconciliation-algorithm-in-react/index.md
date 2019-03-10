> 原文地址：https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e
>
> 原文作者：[Max Koretskyi aka Wizard](https://github.com/maximusk)

![fiber](./assets/fiber.png)

React 是一个用于构建用户界面的 JavaScript 库。它的核心在于能够检测组件状态中的变化并且能将更新状态映射到屏幕上。在 React 中我们把这个过程称作**协调**。当我们调用 `setState` 方法时，React 会检查是否有 `state` 或 `props` 变化并在 UI 上重新渲染组件。

React 文档给出了该协调机制的[高度概括](https://reactjs.org/docs/reconciliation.html)：React 元素、生命周期方法、`render` 方法以及用于组件子项的 diff 算法。从 `render` 方法中返回的不可改变的 React 元素树通常被称作 “virtual DOM” 。这个术语很早就帮助人们理解了 React ，但同时也带来了许多困惑，因而这个术语不再在 React 文档中使用。在本文中，我只会称其为 React 元素树。

除了 React 元素树，该框架中还有用来保持状态的内部实例（组件、DOM 节点等）树。从 React 16 版本开始，React 推出了该内部实例树的全新实现以及用来管理它的 **Fiber** 算法。如果想要了解 Fiber 架构所带来的好处请参考我之前的文章 [React 是怎样和为什么要在 Fiber 中使用链表](https://medium.com/react-in-depth/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-67f1014d0eb7)。

这是本系列的第一篇文章，旨在教会你 React 中的内部架构。在本文中，我想提供一些重要概念以及与 Fiber 算法有关数据结构的深入概述。一旦我们拥有了足够的知识储备，我们将会开始探究该算法的内部原理以及那些用于遍历和处理 fiber 树的重要函数。本系列的下一篇文章将会展示 React 是如何执行初始渲染和处理后续 `state` 与 `props` 更新的。从那里开始我们将继续讨论调度器的细节，即子协调过程，以及构建 `effects` 列表的机制。

我将在这里为你提供一些非常有深度的知识 🧙‍，我鼓励你阅读并理解 Concurrent React 的内部工作原理以及其背后的魔法。如果你想为 React 项目作出自己的贡献，本系列文章也将为你提供很好的指导。我十分热衷于[逆向工程](https://blog.angularindepth.com/level-up-your-reverse-engineering-skills-8f910ae10630)，所以本篇文章中将会有很多关于 React 源码的链接。

本篇文章的内容肯定有很多需要慢慢理解的地方，如果你不能够立即理解其中的某些东西，也不要感到有压力。

虽然会很耗费时间但这一切都是值得的。**请注意你不需要知道任何关于如何使用 React 的内容。本文是关于 React 内部是如何工作的。**

## 引言

下面是一个简单的应用程序，在本系列的文章中我都可能提到它。现在我们有一个可以用来增加屏幕上数字的按钮：

![counter](./assets/counter.gif)

这是它的实现代码：

```jsx
class ClickCounter extends React.Component {
    constructor(props) {
        super(props);
        this.state = {count: 0};
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        this.setState((state) => {
            return {count: state.count + 1};
        });
    }


    render() {
        return [
            <button key="1" onClick={this.handleClick}>Update counter</button>,
            <span key="2">{this.state.count}</span>
        ]
    }
}
```

正如你所见，这是一个简单的组件并且能从 `render` 方法中返回 `button` 和 `span` 两个子元素。当你点击按钮后，组件中的状态就会在内部被更新。反过来，这会导致 `span` 元素中的文本更新。

React 会在**协调**过程中进行很多活动。例如，在组件第一次渲染以及应用程序中的 `state` 更新后 React 会执行相应的高级操作：

* 更新 `ClickCounter` 组件 `state` 中的 `count` 属性。
* 检索并比较 `ClickCounter` 组件的子代和它们的 props
* 为 `span` 元素更新 props

在协调过程中，还有其他例如调用[生命周期方法](https://reactjs.org/docs/react-component.html#updating)和更新 [refs](https://reactjs.org/docs/refs-and-the-dom.html) 的操作执行。**所有的这些活动在 Fiber 架构中被统称为 “work” 。** work 的类型通常取决于 React 元素的类型。例如，对于 class 组件，React 需要创建一个实例，而对于函数组件就不需要这样做。正如你所了解的，在 React 中我们有许多类型的元素。例如 class 组件和函数组件，host 组件（DOM 节点）和 portals 等等。React 元素的类型由 `createElement` 函数的第一个参数定义。这个函数通常在 `render` 方法中用来创建元素。

在我们开始探究 React 中的内部活动和主要的 fiber 算法之前，让我们先熟悉一下 React 内部使用的数据结构。

## 从 React 元素到 Fiber 节点

