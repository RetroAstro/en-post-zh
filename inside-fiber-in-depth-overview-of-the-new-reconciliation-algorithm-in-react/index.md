> 原文地址：https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e
>
> 原文作者：[Max Koretskyi aka Wizard](https://github.com/maximusk)

![fiber](./assets/fiber.png)

React 是一个用于构建用户界面的 JavaScript 库。它的核心在于能够检测组件状态中的变化并且能将更新状态映射到屏幕上。在 React 中我们把这个过程称作**协调**。当我们调用 `setState` 方法时，React 会检查是否有 `state` 或 `props` 变化并在 UI 上重新渲染组件。

React 文档给出了该协调机制的[高度概括](https://reactjs.org/docs/reconciliation.html)：React 元素、生命周期方法、`render` 方法以及用于组件子项的 diff 算法。从 `render` 方法中返回的不可改变的 React 元素树通常被称作 “virtual DOM” 。这个术语很早就帮助人们理解了 React ，但同时也带来了许多困惑，因而这个术语不再在 React 文档中使用。在本文中，我只会称其为 React 元素树。

除了 React 元素树，该框架中还有用来保持状态的内部实例（组件、DOM 节点等）树。从 React 16 版本开始，React 推出了该内部实例树的全新实现以及用来管理它的 **Fiber** 算法。如果想要了解 Fiber 架构所带来的好处请参考我之前的文章 [React 是怎样和为什么要在 Fiber 中使用链表](https://medium.com/react-in-depth/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-67f1014d0eb7)。

这是本系列的第一篇文章，旨在让你理解 React 中的内部架构。在本文中，我想提供一些重要概念以及与 Fiber 算法有关数据结构的深入概述。一旦我们拥有了足够的知识储备，我们将会开始探究该算法的内部原理以及那些用于遍历和处理 fiber 树的重要函数。本系列的下一篇文章将会展示 React 是如何执行初始渲染和处理后续 `state` 与 `props` 更新的。从那里开始我们将继续讨论调度器的细节，即子协调过程，以及构建 `effects` 列表的机制。

我将在这里为你提供一些非常有深度的知识 🧙‍，我鼓励你阅读并理解 Concurrent React 的内部工作原理以及其背后的魔法。如果你想为 React 项目作出自己的贡献，本系列文章也将为你提供很好的指导。我十分热衷于[逆向工程](https://blog.angularindepth.com/level-up-your-reverse-engineering-skills-8f910ae10630)，所以本篇文章中将会有很多关于 React 源码的链接。

本篇文章的内容肯定有很多需要慢慢理解的地方，如果你不能够立即理解其中的某些东西，也不要感到有压力。

虽然会很耗费时间但这一切都是值得的。**请注意你不需要知道任何关于如何使用 React 的内容。本文讲解的是 React 内部是如何工作的。**

## 背景

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

React 中的每个组件都有一个用来展示 UI 的东西，它在 `render` 方法中被返回，通常被叫做视图或者模版。 下面是 `ClickCounter` 组件的模版： 

```jsx
<button key="1" onClick={this.onClick}>Update counter</button>
<span key="2">{this.state.count}</span>
```

### React 元素

一旦模版被 JSX 编译器编译，你就会得到一系列的 React 元素。它们才是 React 组件中 `render` 方法真正所返回的东西，而不是 HTML 。由于我们不一定需要使用 JSX ，故 `ClickCounter` 组件中的 `render` 方法也可以像下面这样重写：

```jsx
class ClickCounter {
    ...
    render() {
        return [
            React.createElement(
                'button',
                {
                    key: '1',
                    onClick: this.onClick
                },
                'Update counter'
            ),
            React.createElement(
                'span',
                {
                    key: '2'
                },
                this.state.count
            )
        ]
    }
}
```

在 `render` 方法中调用的 `React.createElement` 函数会创建像下面这样的两个数据结构：

```js

[
    {
        $$typeof: Symbol(react.element),
        type: 'button',
        key: "1",
        props: {
            children: 'Update counter',
            onClick: () => { ... }
        }
    },
    {
        $$typeof: Symbol(react.element),
        type: 'span',
        key: "2",
        props: {
            children: 0
        }
    }

```

可以看到 React 将 **[$$typeof](https://overreacted.io/why-do-react-elements-have-typeof-property/)** 属性添加到这些对象上，以便用来唯一标识 React 元素。然后我们有像 **`type`** 、**`key`** 和 **`props`** 这些用来描述元素的属性。这些值来自当调用 **`React.createElement`** 函数时传递的参数。请注意 React 是如何将文本内容表示为 **`span`** 和 **`button`** 节点的子项的。还有为什么点击事件会作为 **`button`** 元素中 props 的一部分。在 React 元素中还有其他的一些字段比如 **ref** ，但那已经超出本文所要介绍的内容了。

**`ClickCounter`** 的 React 元素上并没有 props 或者 key ：

```js
{
    $$typeof: Symbol(react.element),
    key: null,
    props: {},
    ref: null,
    type: ClickCounter
}
```

### Fiber 节点

在**协调**期间，从 `render` 方法中返回的每个 React 元素都会被合并到 fiber 节点树中去。每一个 React 元素都有其对应的 fiber 节点。与 React 元素不同的是，fibers 并不会在每次调用 `render` 方法后被重新创建。它们是持有组件状态和 DOM 节点的可变数据结构。

我们之前提到过 React 会根据 React 元素类型的不同来进行不同的活动。在我们的示例应用中，作为 class 组件的 **`ClickCounter`** 会调用生命周期方法以及 `render` 方法，而像 **`span`** 这样的 host 组件（DOM 节点）将会进行 DOM 的改变。因此每个 React 元素都会转换成相应类型的 Fiber 节点，它们用来描述需要被完成的工作。

**你可以将 fiber 想象为有某些工作要做的数据结构，或者换句话说，一个工作单位。Fiber 架构同时也为我们跟踪、调度、暂停和中止工作提供了便利。**

当 React 元素第一次被转换为 fiber 节点，React 通过 [createFiberFromTypeAndProps](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L414) 函数将元素中的数据用来创建 fiber 节点。在后续的更新中 React 会重用 fiber 节点并使用来自相应 React 元素的数据来更新必要的属性。

React 也可能根据 `key` prop 移动层次结构中的节点，或者删除某些 fiber 节点如果相应的 React 元素不再从 `render` 方法中返回的话。

> 想要了解现有 fiber 节点中所执行的所有活动列表和相应的函数，可以查看 [**ChildReconciler**](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactChildFiber.js#L239) 函数。

因为 React 会为每个 React 元素创建相应的 fiber 节点，既然我们有 React 元素树，那我们也会有对应的 fiber 节点树。在这种情况下，我们的示例应用会像下面这样展示其 fiber 结构：

![sample](./assets/sample.png)

所有的 fiber 节点都通过链表连接，每个节点上都会有 **`child`** 、**`sibling`** 和 **`return`** 属性。如果想要理解为什么要这样设计，你可以参考我之前的文章 [React 是怎样和为什么要在 Fiber 中使用链表](https://medium.com/react-in-depth/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-67f1014d0eb7) 。

### current 树 与 work in progress 树

在第一次渲染结束后，React 会生成一棵将应用中的 state 映射到 UI 上的 fiber 节点树。而这棵树通常被称作 **current** 树。当 React 开始执行更新渲染时会创建一棵用于将未来的 state 映射到屏幕上的 **workInProgress** 树。

所有在 fibers 上进行的工作都会在 **`workInProgress`** 树中执行。当 React 遍历 **`current`** 树时，会为每个存在的节点创建一个构成 **`workInProgress`** 树的备用节点。该节点是通过 `render` 方法返回的 React 元素数据创建的。

当更新和相应的工作执行完毕后，React 就会拥有一棵备用树且随时准备映射到屏幕上。一旦 **`workInProgress`** 树在屏幕上被渲染，它也就变成了 **`current`** 树。

React 的核心原则之一是一致性。React 总是一次更新 DOM  — 它不会显示部分的结果。**`workInProgress`** 树就像草稿一样，对用户是不可见的，因此 React 可以先处理完所有组件，然后再将更新一次性映射到屏幕上。

在源码中你可以看见许多函数都需要从 **`current`** 和 **`workInProgress`** 树中获取 fiber 节点，比如下面这个函数：

```js
function updateHostComponent(current, workInProgress, renderExpirationTime) {...}
```

每个 fiber 节点在 **alternate** 字段中都保留着对应的另一棵树上节点的引用。例如 **`current`** 树上的节点指向 **`workInProgress`** 树中对应的节点，反之亦然。

### 副作用

我们可以把 React 组件想象成一个用 state 和 props 来计算最终 UI 是如何展示的函数。每个像操作 DOM 节点或调用生命周期方法的活动都应该被称作副作用又或是一次影响。[副作用](https://reactjs.org/docs/hooks-overview.html#%EF%B8%8F-effect-hook)在 React 文档中也被提到过。

> 在以前，你可能会在 React 组件中进行像数据获取、订阅数据源或者手动操作 DOM 等活动。我们把这些操作称为 “副作用” 因为它们会影响其他的组件并且在渲染过程中无法完成。

如你所见，大多数的 state 和 props 更新都将导致 “副作用” 的发生。既然执行副作用是一种类型的工作，那 fiber 节点除了用于更新外，它也可以作为跟踪 effects 的便利机制。每个 fiber 节点都有与之对应的 effects 。它们在 **`effectTag`** 字段中被编码。

因此 effects 代表着在处理更新后需要为实例完成的[工作](https://github.com/facebook/react/blob/b87aabdfe1b7461e7331abb3601d9e6bb27544bc/packages/shared/ReactSideEffectTags.js) 。对于 host 组件（DOM 节点）来说可能包含增加、更新和删除元素的工作。而对于 class 组件 React 可能会更新 refs 以及调用像 `componentDidMount` 和 `componentDidUpdate` 这样的生命周期方法。当然也有其他的 effects 对应着其他类型的 fiber 。

### Effects 列表

React 能够非常快速地进行渲染更新，为了达到这样的高性能水平，它采用了一些有趣的技术。**其中之一就是用线性表构建带有 effects 的 fiber 节点，这样就拥有了快速迭代的效果。** 迭代线性表远比迭代树要来的快，我们没有必要在不含副作用的节点上浪费时间。

该列表的目标是标记具有 DOM 更新或者与 effects 相关的节点。该列表是 **`finishedWork`** 树的子集，使用 **`nextEffects`** 属性而不是 **`current`** 和 **`workInProgress`** 树中的 **`child`** 属性进行连接。

[Dan Abramov](https://medium.com/@dan_abramov) 为 effects 列表给出了一个类比。他喜欢将其比作圣诞树，所有的 effects 节点都会被 “圣诞灯” 绑定在一起。为了将其可视化，让我们想象一下在含有 fiber 节点的树中，那些高亮着的节点就代表着需要被完成的工作。例如，我们的更新导致 **`c2`** 被插入 DOM 中，**`d2`** 和 **`c1`** 会改变其自身的属性，而 **`b2`** 则要调用某个生命周期方法。effects 列表会将它们连接起来，便于 React 在之后直接跳过其他节点：

![linear](./assets/linear.png)

现在你应该明白具有 effects 的节点是如何连接在一起的了。当我们遍历节点时，React 用 **`firstEffect`** 指针来找到列表的开头。因此上图也可以表示为这样的线性表：

![list](./assets/list.png)

如你所见，React 会按照从子代到双亲的顺序依次调用 effects。

### Fiber 树的头节点

每一个 React 应用都会有一个或者多个 DOM 元素作为容器。在本例中指的是 ID 为 **`container`** 的 **`div`** 元素。

```js
const domContainer = document.querySelector('#container')
ReactDOM.render(React.createElement(ClickCounter), domContainer)
```

React 会为每一个作为容器的元素创建一个 [fiber root](https://github.com/facebook/react/blob/0dc0ddc1ef5f90fe48b58f1a1ba753757961fc74/packages/react-reconciler/src/ReactFiberRoot.js#L31) 对象，你可以通过对 DOM 元素的引用来访问它。

```js
const fiberRoot = query('#container')._reactRootContainer._internalRoot
```

fiber root 保留着对 fiber 树的引用。它被储存在 fiber root 的 **`current`** 属性上： 

```js
const hostRootFiberNode = fiberRoot.current
```

fiber 树的头节点是一种[特殊类型](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/shared/ReactWorkTags.js#L34)的 fiber 节点，它被称作 **`HostRoot`** 。它在内部被创建，并充当了最顶层组件的父级。**`HostRoot`** fiber 节点上有一个 **`stateNode`** 属性用于返回到 **`FiberRoot`** ：

```js
fiberRoot.current.stateNode === fiberRoot; // true
```

你可以通过 fiber root 来访问最顶层的 **`HostRoot`** fiber 节点，然后再进一步地深入 fiber 树。或者你也可以直接从组件实例上获取独立的 fiber 节点就像下面这样：

```js
compInstance._reactInternalFiber
```

### Fiber 节点的结构

让我们来看下为 **`ClickCounter`** 组件创建的 fiber 节点的结构：

```js
{
    stateNode: new ClickCounter,
    type: ClickCounter,
    alternate: null,
    key: null,
    updateQueue: null,
    memoizedState: {count: 0},
    pendingProps: {},
    memoizedProps: {},
    tag: 1,
    effectTag: 0,
    nextEffect: null
}
```

这是 **`span`** DOM 元素的 fiber 结构：

```js
{
    stateNode: new HTMLSpanElement,
    type: "span",
    alternate: null,
    key: "2",
    updateQueue: null,
    memoizedState: null,
    pendingProps: {children: 0},
    memoizedProps: {children: 0},
    tag: 5,
    effectTag: 0,
    nextEffect: null
}
```

fiber 节点上有很多的字段。在之前我已经介绍过 **`alternate`** 、**`effectTag`** 和 **`nextEffect`** 字段的作用。下面让我们开始理解为什么还需要其他的字段吧。

#### stateNode

持有对组件的类实例、DOM 节点或者其他与 fiber 节点相关的 React 元素类型。一般来讲，我们可以说这个属性是用来保存与 fiber 相关的局部状态。

#### type

定义 fiber 是函数还是类。对于 class 组件来讲，它指向 constructor 函数而对于 DOM 元素来讲它用来描述一个 HTML 标签。我经常用这个字段来理解 fiber 节点到底与哪个 React 元素相关联。

#### tag

定义 [fiber 的类型](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/shared/ReactWorkTags.js)。它在协调算法中用于确定那些需要完成的 work 。就像之前提到过的，这些 work 会因为 React 元素类型的不同而不同。[createFiberFromTypeAndProps](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L414) 函数会将 React 元素映射到相应的 fiber 节点类型上。在我们的应用中，**`ClickCounter`** 组件的 **`tag`** 是 **`1`** 代表着 **`ClassComponent`** ，而 **`span`** 元素为 **`5`** 代表着 **`HostComponent`** 。

#### updateQueue

用于 state 更新、回调和 DOM 更新的队列。

#### memoizedState

用来创建输出的 fiber 节点上的 state 。在处理更新时，它反映着将会在当前屏幕上渲染的 state 。

#### memoizedProps

在前一次渲染过程中用来创建输出的 fiber 节点上的 props 。

#### pendingProps

从 React 元素的新数据中更新的 props ，等待着在其子组件和 DOM 元素中被使用。

#### key

一组子代中的唯一标识符，能够帮助 React 确定列表中哪些节点发生了改变、增加或删除。

你可以在[这里](https://github.com/facebook/react/blob/6e4f7c788603dac7fccd227a4852c110b072fe16/packages/react-reconciler/src/ReactFiber.js#L78)找到完整的 fiber 节点结构。我在上面的解释中省略了一些字段。特别的，我跳过了 **`child`** 、**`sibling`** 和 **`return`** 这些指针，它们构成了[上篇文章](https://medium.com/react-in-depth/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-67f1014d0eb7)中我所描述的一种树数据结构。还有像 **`expirationTime`** 、**`childExpirationTime`** 和 **`mode`** 这些与 **`Scheduler`** 特定类别相关的字段。

## 通用算法

React 主要在两个阶段执行相应的 work ：**render** 阶段和 **commit** 阶段。

在第一次 **render** 阶段执行期间，React 会将更新应用于那些通过 **`setState`** 或者 **`React.render`** 调度的组件，并且判断出需要在 UI 中更新的内容。如果是初次渲染，React 会为从 **`render`** 方法中返回的每一个元素创建一个新的 fiber 节点。在后续的更新中，已有的 React 元素对应的 fiber 节点会被重用和更新。**这个阶段最终的执行结果就是生成一棵带有副作用的 fiber 节点树。** 这些 effects 描述了在 **`commit`** 阶段将会被执行的 work 。在 **`commit`** 阶段，React 会将带有副作用的 fiber 节点树作用到实例上。它会遍历整个 effects 列表然后执行像 DOM 更新和其他对用户可见的操作。

**理解在 `render` 阶段执行的工作可以是异步的十分重要。** React 能够根据可用时间同时处理一个或者多个 fiber 节点，如果发现本轮的可用时间耗尽，React 就会先停止下来把那些已经完成的 work 存好然后暂停某些事件。当又有可用时间的时候才会从原先停止的地方继续。虽然在有些时候 React 不得不丢弃已经完成的 work 然后再从头开始。我们之所以在 **`render`** 阶段能够暂停那些正在执行的 work 是因为这些 work 并不会造成用户可见的改变，例如 DOM 的更新。**相反，接下来的 `commit` 阶段总是同步执行的。** 因为在这个阶段的操作会直接造成对用户可见的改变，比如说 DOM 的更新。这也正是 React 需要一次就完成它们的原因。

调用生命周期方法是 React 执行 work 的类型之一。有些方法在 **`render`** 阶段被调用，而另外一些方法则会在 **`commit`** 阶段被调用。下面是在 **`render`** 阶段会被调用的生命周期方法列表：

- [UNSAFE_] componentWillMount (弃用)
- [UNSAFE_] componentWillReceiveProps (弃用)
- getDerivedStateFromProps
- shouldComponentUpdate
- [UNSAFE_] componentWillUpdate (弃用)
- render 

如你所见，在 **`render`** 阶段执行的一些遗留的生命周期方法从 React 16.3 版本开始就已经被标记为 **`UNSAFE`** 。这些遗留的生命周期方法将会在 React 16.x 版本中被弃用，而与之同名但没有被标记的生命周期方法将会在 17.0 版本中被移除。

你会对 React 团队做出这样的改变而感到好奇吗？

好吧，我们刚刚才了解到在 **`render`** 阶段并不会进行像 DOM 更新那样的副作用操作，因此 React 就有了处理异步更新与异步组件的能力（甚至能够以多线程的方式进行此操作）。然而，那些标有 **`UNSAFE`** 的生命周期方法经常会被误解和误用。开发者会把带有副作用的代码放入这些生命周期方法中，而在新的异步渲染机制里这样做可能会出现问题。虽然那些同名但没有被标记为 **`UNSAFE`** 的生命周期方法将会在未来被移除，但在即将来临的并发模式下的 React 中使用这些 **`UNSAFE`** 标记的生命周期方法仍会有很大几率出现 bug 。

下面是在 **`commit`** 阶段会执行的生命周期方法列表：

- getSnapshotBeforeUpdate
- componentDidMount
- componentDidUpdate
- componentWillUnmount

因为这些方法在同步的 **`commit`** 阶段被执行，因而它们中可能带有副作用并且也可能进行 DOM 的操作。

好了，现在我们拥有了必要的前置知识，这足以让我们深入用于遍历树和执行 work 的通用算法。让我们开始吧。

### Render 阶段

协调算法总是使用 [renderRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1132) 函数从最顶层的 **`HostRoot`** fiber 节点开始。但是，React 会跳过那些已经处理过的 fiber 节点直到发现带有未完成的 work 的 fiber 节点。例如，如果你在组件树的深处调用 **`setState`** ，React 会从最顶层开始但它会很快地跳过那些父组件直到发现调用 **`setState`** 方法的组件。

#### work loop 的主要步骤

所有的 fiber 节点都会在 [work loop](https://github.com/facebook/react/blob/f765f022534958bcf49120bf23bc1aa665e8f651/packages/react-reconciler/src/ReactFiberScheduler.js#L1136) 中被处理。下面是该 loop 中同步执行部分的实现代码：

```js
function workLoop(isYieldy) {
  if (!isYieldy) {
    while (nextUnitOfWork !== null) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    }
  } else {...}
}
```

在上面的代码中，**`nextUnitOfWork`** 持有来自 **`workInProgress`** 树中那些带有 work 且需要被执行的 fiber 节点的引用。当 React 遍历整个 Fiber 树时，通过这个变量就能够清楚地知道哪些 fiber 节点上含有未完成的 work 。在当前的 fiber 被处理完之后，这个变量要么会持有下一个树中的 fiber 节点的引用要么就为 **`null`** 。在这种情况下 React 便会退出 work loop 然后随时准备着 commit 已有的改变。

在这里我们有四种主要的函数用于遍历树以及开始或结束某个 work ：

- [performUnitOfWork](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1056)
- [beginWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L1489)
- [completeUnitOfWork](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L879)
- [completeWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberCompleteWork.js#L532)

为了展示它们是如何被使用的，请看下面用来遍历一棵 fiber 树的动画。为了展示这个例子我给出了这些函数的简单实现。每个函数都需要处理相应的 fiber 节点，当 React 往下遍历时，你可以看到当前活动的 fiber 节点发生了改变。你可以在下面我给出的视频链接中清晰地看出这个算法在遍历树时是如何从一个分支切换到另一个分支上去的。它首先会完成子 fiber 节点中带有的 work ，然后才会移动到父 fiber 节点上。

![works](./assets/works.gif)

> ***请注意由直线垂直连接的 fiber 节点属于兄弟节点，而以弯曲方式连接的节点才是父子节点的关系，例如 `b1` 没有子节点，而 `b2` 却有一个叫做 `c1` 的子节点。***

[这是视频的链接](https://vimeo.com/302222454)，你可以随时暂停播放并检查当前的节点与函数执行的状态。从概念上来讲，你可以将 “begin” 看作刚进入组件并开始相应的 work ，而 “compete” 则代表着 work 执行完毕可以退出组件。你也可以先看看我[给出的实现代码和相应的例子](https://stackblitz.com/edit/js-ntqfil?file=index.js)，因为马上我将会解释这些函数的作用。

让我们先从最开始的 **`performUnitOfWork`** 和 **`beginWork`** 函数开始：

```js
function performUnitOfWork(workInProgress) {
    let next = beginWork(workInProgress);
    if (next === null) {
        next = completeUnitOfWork(workInProgress);
    }
    return next;
}

function beginWork(workInProgress) {
    console.log('work performed for ' + workInProgress.name);
    return workInProgress.child;
}
```

**`performUnitOfWork`** 函数接收来自 **`workInProgress`** 树中的 fiber 节点，然后用 **`beginWork`** 函数开始 work 的执行。这个函数会开始当前 fiber 中所有需要被执行的活动。出于展示的目的，我们只是简单地打印了当前 fiber 的名字来表示已经完成的 work 。**`beginWork` 函数总是会返回在 loop 中需要被处理的下一个子 fiber 节点或者是 `null` 。** 

如果有下一个子节点，那么它将在 **`workLoop`** 函数中被赋值给 **`nextUnitOfWork`** 变量。然而，要是没有下一个子节点，React 就会察觉到此时到达了分支的底部因此便可以结束当前节点的 work 。**一旦当前节点的 work 结束，React 便会开始执行它的兄弟节点上的 work，然后再返回到父节点上。** 下面的代码是在 **`completeUnitOfWork`** 函数中完成的：

```js
function completeUnitOfWork(workInProgress) {
    while (true) {
        let returnFiber = workInProgress.return;
        let siblingFiber = workInProgress.sibling;

        nextUnitOfWork = completeWork(workInProgress);

        if (siblingFiber !== null) {
            // If there is a sibling, return it
            // to perform work for this sibling
            return siblingFiber;
        } else if (returnFiber !== null) {
            // If there's no more work in this returnFiber,
            // continue the loop to complete the parent.
            workInProgress = returnFiber;
            continue;
        } else {
            // We've reached the root.
            return null;
        }
    }
}

function completeWork(workInProgress) {
    console.log('work completed for ' + workInProgress.name);
    return null;
}
```

你可以看到这个函数由很大的 **`while`** 循环构成。当 **`workInProgress`** 树中的节点没有子节点的时候，React 便会执行上面这个函数。在结束当前的 fiber 节点后，便会检查该节点是否含有兄弟节点。如果有兄弟节点，React 便会提前跳出循环并返回该兄弟节点。返回的这个节点将会被赋值给 **`nextUnitOfWork`** 变量，然后 React 便会从这个兄弟节点开始的分支往下继续执行相应的 work 。理解 React 只会先完成前面兄弟节点的 work 十分重要，它并不会提前结束父节点的 work 。**只有在完成以子节点开始的所有分支上的 work 之后，React 才会结束父节点上的 work 然后再进行回溯的工作。** 

从上面的实现代码可以看出，**`performUnitOfWork`** 与 **`completeUnitOfWork`** 函数主要用于迭代。而真正的活动发生在 **`beginWork`** 和 **`completeWork`** 函数中。在本系列剩下的文章中我们会了解到当 React 执行 **`beginWork`** 与 **`completeWork`** 时我们的 **`ClickCounter`** 组件和 **`span`** 节点具体发生了什么。

###  Commit 阶段

这个阶段以调用 [completeRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L2306) 函数作为开始。在这个阶段 React 会执行更新 DOM 的操作以及调用可引起突变的 pre 与 post 钩子生命周期方法。

在 React 进入这个阶段时，它拥有着两棵树和 effects 列表。第一棵树代表着此刻在屏幕上渲染的状态。而另一棵则是在 **`render`** 阶段生成的备用树。在源码中它通常被叫做 **`finishedWork`** 或者 **`workInProgress`** 树。该备用树与当前树一样，通过 **`child`** 与 **`sibling`** 指针连接。

接下来，我们还有一个 effects 列表  — 来自 **`finishedWork`** 树且通过 **`nextEffect`** 指针连接的一组节点子集。需要记住的是 effects 列表是执行完 **`render`** 阶段后的结果。渲染的最终目的是要确定哪些 DOM 节点应该被插入、更新或者删除，哪些组件上的生命周期方法需要被调用。这便是 effects 列表要告诉我们的东西，**同时它们也是在 commit 阶段将会被迭代的节点集。** 

> ***出于方便调试的目的，我们可以通过 fiber root 上的 `current` 属性拿到  `current` 树。同时我们也能通过在 `current` 树中 `HostFiber` 上的 `alternate` 属性拿到 `finishedWork` 树。***

在 commit 阶段主要运行的函数是 [commitRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L523) 。

它大致会执行以下的工作：

* 在标有 **`Snapshot`** effect 的节点上调用 **`getSnapshotBeforeUpdate`** 生命周期方法。
* 在标有 **`Deletion`** effect 的节点上调用 **`componentWillUnmount`** 生命周期方法。
* 执行 DOM 的插入、更新和删除操作。
* 将 **`finishedWork`** 树标记为 **`current`** 树。
* 在标有 **`Placement`** effect 的节点上调用 **`componentDidUpdate`** 生命周期方法。

在调用突变前的 **`getSnapshotBeforeUpdate`** 生命周期方法后，React 会提交树中产生的所有副作用。它将通过两轮进行，第一轮会执行所有 DOM (host) 的插入、更新、删除以及 ref 的卸载。之后 React 会把 **`finishedWork`** 树交给 **`FiberRoot`** 并将 **`workInProgress`** 树标记为 **`current`** 树。这是在 commit 阶段的第一轮之后要完成的工作，这样在调用 **`componentWillUnmount`** 生命周期方法时引用的仍然是先前的那棵 **`current`** 树，但在 commit 阶段的第二轮开始之前，**`finishedWork`** 树已经被标记为 **`current`** 树，因此在调用 **`componentDidMount/Update`** 生命周期方法时引用的就是最新的 **`current`** 树。在第二轮进行的时候，React 会调用所有其他的生命周期方法以及 ref 相关的回调。这些生命周期方法被放在单独的一轮中进行为的就是确保在这之前树中所有的插入、更新和删除操作都已经执行完毕。

下面是上述步骤中函数运行的重点：

```js
function commitRoot(root, finishedWork) {
    commitBeforeMutationLifecycles()
    commitAllHostEffects();
    root.current = finishedWork;
    commitAllLifeCycles();
}
```

代码中的每一个子函数都实现了用于遍历 effects 列表并查看相应 effects 类型的方法，当发现与该函数目的相关的 effect 时，React 便会立刻调用它。

#### 突变前的生命周期方法

例如，下面的代码会遍历整个 effects 树并检查某个带有 **`Snapshot`** effect 的节点：

```js
function commitBeforeMutationLifecycles() {
    while (nextEffect !== null) {
        const effectTag = nextEffect.effectTag;
        if (effectTag & Snapshot) {
            const current = nextEffect.alternate;
            commitBeforeMutationLifeCycles(current, nextEffect);
        }
        nextEffect = nextEffect.nextEffect;
    }
}
```

对于 class 组件来讲，这里的 effect 就代表着调用 **`getSnapshotBeforeUpdate`** 生命周期方法。

#### DOM 的更新

[commitAllHostEffects](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L376) 函数是 React 执行 DOM 更新的地方。该函数基本上定义了需要为节点完成和执行的操作类型：

```js
function commitAllHostEffects() {
    switch (primaryEffectTag) {
        case Placement: {
            commitPlacement(nextEffect);
            ...
        }
        case PlacementAndUpdate: {
            commitPlacement(nextEffect);
            commitWork(current, nextEffect);
            ...
        }
        case Update: {
            commitWork(current, nextEffect);
            ...
        }
        case Deletion: {
            commitDeletion(nextEffect);
            ...
        }
    }
}
```

有趣的是，React 把调用 **`componentWillUnmount`** 的生命周期方法作为了 **`commitDeletion`** 函数中处理删除操作的一部分来进行。

#### 突变后的生命周期方法

[commitAllLifecycles](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L465) 函数则是 React 调用剩下的像 **`componentDidUpdate`** 和 **`componentDidMount`** 等生命周期方法的地方。

