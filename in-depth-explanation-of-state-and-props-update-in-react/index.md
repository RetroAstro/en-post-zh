> 原文地址：https://medium.com/react-in-depth/in-depth-explanation-of-state-and-props-update-in-react-51ab94563311
>
> 原文作者：[Max Koretskyi aka Wizard](https://github.com/maximusk)

![](./assets/state-props.png)

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

## 调度更新

当我们点击 button 的时候，**`click`** 事件被触发，之后 React 就会执行我们传入 button props 中的回调函数。在我们的示例程序中只是简单地增加计数并且更新 state ：

```js
class ClickCounter extends React.Component {
    ...
    handleClick() {
        this.setState((state) => {
            return {count: state.count + 1};
        });
    }
} 
```

每一个 React 组件都有其对应的 **`updater`** 作为该组件与 React core 之间的桥梁。它让 React DOM ，React Native ，服务端渲染以及测试工具能够以不同的方式实现 **`setState`** 方法。

在本篇文章中，我们会了解到 React DOM 中的 updater 对象是如何实现的，其实它的核心就是 Fiber 协调器。对于 **`ClickCounter`** 组件来说对应的是 [**`classComponentUpdater`**](https://github.com/facebook/react/blob/6938dcaacbffb901df27782b7821836961a5b68d/packages/react-reconciler/src/ReactFiberClassComponent.js#L186) 。它主要负责检索 Fiber 实例，将更新放入队列以及调度相应的 work 。

当 state 更新在排队时，它们基本上就只是在等待被添加到 Fiber 节点需要执行的更新队列里去。在我们的例子中，对应着 **`ClickCounter`** 组件的 [Fiber 节点](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e) 会有如下的结构：

```js
{
    stateNode: new ClickCounter,
    type: ClickCounter,
    updateQueue: {
         baseState: {count: 0}
         firstUpdate: {
             next: {
                 payload: (state) => { return {count: state.count + 1} }
             }
         },
         ...
     },
     ...
}
```

如你所见，在 **`updateQueue.firstUpdate.next.payload`** 里的函数是我们在 **`ClickCounter`** 组件中为 **`setState`** 方法传入的回调。它代表着第一次更新时需要在 **`render`** 阶段调用的函数。

## 处理 ClickCounter Fiber 节点中的更新

在先前的文章中我提到过 [work loop](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e) ，并且解释了全局变量 **`nextUnitOfWork`** 的作用。需要注意的是，这个变量保留着来自 **`workInProgress`** 树且带有 work 的 Fiber 节点的引用。当 React 在遍历整棵 Fiber 树时，它会根据这个变量来判断是否还有携带着未完成 work 的 Fiber 节点。

我们假设此时 **`setState`** 方法已经被调用。React 会将 **`setState`** 中的回调推入 **`ClickCounter`** Fiber 节点的 **`updateQueue`** 并开始调度相应的 work 。React 此时会进入 **`render`** 阶段。通过调用 [renderRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1132) 函数，它从最顶层的 **`HostRoot`** 节点开始遍历整棵 Fiber 树。然而，在这个过程中 React 会跳过那些已经被处理过的节点直到找到带有未完成 work 的 Fiber 节点。而此时我们仅有一个带有 work 的 Fiber 节点。它就是 **`ClickCounter`** Fiber 节点。

## beginWork

首先，让我们先来理解 [beginWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L1489) 函数。

> *因为在树中的每个 Fiber 节点里都会调用这个函数，所以如果你想在 **`render`** 阶段进行调试，那在这个函数中打断点是最合适不过了。我经常这样做并且会检查相应 Fiber 节点的类型以便找到我需要的那个。*

**`beginWork`** 函数基本上是由一个大的 `switch` 语句构成，它决定了对于标记过的 Fiber 节点来说哪种类型的 work 是需要被完成的，之后便会执行相应的函数去完成 work 。在我们的 **`ClickCounter`** 例子中它是一个 class 组件，所以该分支大概会像下面这样：

```js
function beginWork(current$$1, workInProgress, ...) {
    ...
    switch (workInProgress.tag) {
        ...
        case FunctionalComponent: {...}
        case ClassComponent:
        {
            ...
            return updateClassComponent(current$$1, workInProgress, ...);
        }
        case HostComponent: {...}
        case ...
}
```

让我们再深入到 [**`updateClassComponent`**](https://github.com/facebook/react/blob/1034e26fe5e42ba07492a736da7bdf5bf2108bc6/packages/react-reconciler/src/ReactFiberBeginWork.js#L428) 函数中去。取决于该组件是否为第一次渲染，重新恢复 work 还是一次 state 更新，React 要么创建一个实例并将其挂载到组件上要么就只进行 state 更新：

```js
function updateClassComponent(current, workInProgress, Component, ...) {
    ...
    const instance = workInProgress.stateNode;
    let shouldUpdate;
    if (instance === null) {
        ...
        // 在初次渲染时我们需要创建 class 实例
        constructClassInstance(workInProgress, Component, ...);
        mountClassInstance(workInProgress, Component, ...);
        shouldUpdate = true;
    } else if (current === null) {
        // 在恢复执行时，直接重用之前的 class 实例
        shouldUpdate = resumeMountClassInstance(workInProgress, Component, ...);
    } else {
        shouldUpdate = updateClassInstance(current, workInProgress, ...);
    }
    return finishClassComponent(current, workInProgress, Component, shouldUpdate, ...);
}
```

