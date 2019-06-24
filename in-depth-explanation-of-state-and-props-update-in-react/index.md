> 原文地址：https://medium.com/react-in-depth/in-depth-explanation-of-state-and-props-update-in-react-51ab94563311
>
> 原文作者：[Max Koretskyi aka Wizard](https://github.com/maximusk)

在我先前的文章[深入理解 React 中的新协调算法](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e)中，我讲述了很多重要的内容，这为理解我将在本篇文章中提及的 React 更新机制奠定了基础。

我已经概述过将要在本文中使用的主要数据结构和概念，特别是 Fiber 节点，current 与 work in progress 树，副作用以及 effects 列表。我也对协调的主要算法进行过高度概括，而且还讲解了 **`render`** 和 **`commit`** 两个阶段的不同。如果你还没有读过这篇文章，那我建议你从这里开始。

同样的，我仍然会以能够在屏幕上增加数字的按钮作为示例程序：

![](./assets/counter.gif)

你可以在线尝试这个[程序](https://stackblitz.com/edit/react-jwqn64)。我们实现了一个能够通过 **`render`** 方法返回两个子元素 **`button`** 和 **`span`** 的简单组件。当你点击按钮的时候，组件中的 state 会在处理程序内部更新。同时 **`span`** 元素中的文本内容也会更新：

```js

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
    
    componentDidUpdate() {}

    render() {
        return [
            <button key="1" onClick={this.handleClick}>Update counter</button>,
            <span key="2">{this.state.count}</span>
        ]
    }
}
```

在这里，我还将 **`componentDidUpdate`** 生命周期方法添加到了组件中。这是为了展示在 **`commit`** 阶段，React 是如何通过添加副作用来调用该方法的。

在本篇文章中我想向你展示 React 是如何执行 state 更新以及它是如何构建 effects 列表的。我们将会深入理解 **`render`** 阶段与 **`commit`** 阶段中的高级函数。

特别的，我们将在 React 源码的 [**`completeWork`**](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberCompleteWork.js#L532) 函数中看到：

* 在 **`ClickCounter`** 组件的 **`state`** 中更新 **`count`** 属性
* 调用 **`render`** 方法以得到子元素列表然后进行协调比较
* 更新 **`span`** 元素中的 props

在 React 源码的 [**`commitRoot`**](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L523) 函数中：

* 更新 **`span`** 元素中的 **`textContent`** 属性
* 调用 **`componentDidUpdate`** 生命周期方法

但在这之前，让我们先快速理解一遍在 click 事件中调用 **`setState`** 方法后，React 是如何执行相应的工作的。

**请注意你并不需要知道该如何使用 React 。本篇文章讲解的是 React 的内部工作原理。**
