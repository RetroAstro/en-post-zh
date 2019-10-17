> 原文地址：https://blog.logrocket.com/a-complete-guide-to-threads-in-node-js-4fa3898fe74f/
>
> 原文作者：[Maciej Cieślar](https://github.com/maciejcieslar) 

很多人想知道基于单线程的 [Node.js](https://nodejs.org/en/) 是如何与多线程的后端竞争的。同时，考虑到 Node.js 单线程的性质，许多大型公司却将 Node 作为他们的后端选型，这似乎是违背常理的。想知道为什么会这样做，我们就得理解当我们在讲 Node.js 单线程时，它的真正含义是什么。

在 JavaScript 诞生之初，它的主要用途在于能够完成 web 上的简单操作，例如验证表单或者创建一个彩虹色的鼠标轨迹。直到 2009 年，Node.js 的创造者 Ryan Dahl ，让开发者能够使用 JavaScript 来编写后端应用成为可能。

一般的后端语言，都会支持多线程机制，并且具有许多用于在线程与其他面向线程特性之间同步值的各种机制。如果将这些特性添加到 JavaScript 中去，估计就得重写整门语言，显然这不是 Dahl 想要的结果。为了让 JavaScript 支持多线程机制，就必须得想出合适的解决办法，下面就让我们来解释 Node.js 到底是怎样工作的。

Node.js 工作原理
-----

Node.js 采用了两类线程：由事件循环 ( event loop ) 管理的主线程以及在工作池 ( worker pool ) 中的辅助线程。

事件循环是一种接收回调函数并将其注册以便在未来执行的机制。它与正常的 JavaScript 代码共享一个线程，当 JavaScript 代码阻塞主线程时，事件循环也会被阻塞。

工作池是一个执行模型，用来生成并处理单独的线程。它会同步地执行任务并将最终结果返回给事件循环。然后，事件循环使用该结果来执行相应的回调函数。

