> 原文地址：https://blog.logrocket.com/a-complete-guide-to-threads-in-node-js-4fa3898fe74f/
>
> 原文作者：[Maciej Cieślar](https://github.com/maciejcieslar) 

很多人想知道基于单线程的 [Node.js](https://nodejs.org/en/) 是如何与多线程的后端竞争的。同时，考虑到 Node.js 单线程的性质，许多大型公司却将 Node 作为他们的后端选型，这似乎是违背常理的。想知道为什么会这样做，就得理解当我们在讲 Node.js 单线程时，它的真正含义是什么。

在 JavaScript 诞生之初，它的主要用途在于能够完成 web 上的一些简单操作，例如验证表单或者创建一个彩虹色的鼠标轨迹。直到 2009 年，Node.js 的创造者 Ryan Dahl ，让开发者能够使用 JavaScript 来编写后端应用成为可能。

一般的后端语言，都会支持多线程机制，并且具有许多用于在线程与其他面向线程特性之间同步值的各种机制。如果将这些特性添加到 JavaScript 中去，估计就得重写整门语言，显然这不是 Dahl 想要的结果。为了让 JavaScript 支持多线程机制，就必须得想出合适的解决办法，下面就让我们来解释 Node.js 到底是怎样工作的。

Node.js 工作原理
-----

Node.js 采用了两类线程：由事件循环 ( event loop ) 管理的主线程以及在工作池 ( worker pool ) 中的辅助线程。

事件循环是一种接收回调函数并将其注册以便在未来执行的机制。它与正常的 JavaScript 代码共享一个线程，当 JavaScript 代码阻塞主线程时，事件循环也会被阻塞。

工作池是一个执行模型，用来生成并处理单独的线程。它会同步地执行任务并将最终结果返回给事件循环。然后，事件循环会使用该结果来执行相应的回调函数。

简而言之，事件循环主要负责异步 I / O 操作 — 包括读写磁盘与网络请求等交互。它在 `fs` ( I / O ) 和 `crypto` ( CPU ) 等模块中有广泛地应用。工作池是由 [libuv](http://docs.libuv.org/en/v1.x) 这个库实现的，这也就意味着每当节点需要在 JavaScript 与 C++ 之间进行通信时，就会导致轻微的延迟，不过这点很难被察觉到。

有了上面的这些机制，我们就能够像这样写代码：

```js
fs.readFile(path.join(__dirname, './package.json'), (err, content) => {
 if (err) {
   return null
 }
 console.log(content.toString())
})
```

上面的 `fs` 模块告诉工作池使用它的一个线程来读取文件内容并在完成时通知事件循环。事件循环则找到对应的回调函数，将文件内容作为其参数传入并执行该函数。

上面是一个非阻塞代码的例子，因此，我们不必同步地等待结果返回。我们告诉工作池去读取文件内容，之后再用返回内容调用提供的回调函数。由于工作池有自己的线程，所以在读取文件时，事件循环仍能持续地工作。

当我们需要同步地执行某些复杂操作（例如某个函数执行耗时太长）时，事件循环机制便没有那么地管用了。如果应用中有太多耗时的函数，就可能会显著地降低服务器吞吐量，或者完全冻结它。在这种情况下，我们更没有办法将异步任务委托给工作池。

在那些需要复杂计算的领域 —  例如 AI 、机器学习或者大数据，由于计算操作阻塞了主线程（且是仅有线程），使得服务器无法响应，因此便无法真正有效地使用 Node.js 。直到 Node.js v10.5.0 版本之后，才真正有了对多线程的支持。

介绍工作线程
-----

`worker_threads` 模块能够让我们创建功能完备的多线程 Node.js 应用。

线程工作程序 ( thread worker ) 是在单独线程中生成的一段代码（通常为一个单独的文件）

想要使用线程工作程序，我们就得引入 `worker_threads` 模块。首先让我们来创建一个能够帮助我们生成线程工作程序的函数，然后再讲讲它的内部实现。

```ts
type WorkerCallback = (err: any, result?: any) => any

export function runWorker(path: string, cb: WorkerCallback, workerData: object | null = null) {
 const worker = new Worker(path, { workerData })
 worker.on('message', cb.bind(null, null))
 worker.on('error', cb)
 worker.on('exit', (exitCode) => {
   if (exitCode === 0) {
     return null;
   }
   return cb(new Error(`Worker has stopped with code ${exitCode}`))
 })
 return worker
}
```

为了创建线程工作程序，我们就得生成 `Worker` 类的实例。该函数的第一个参数是包含线程工作程序代码的文件路径，在第二个参数中我们提供了一个包含属性名为 `workerData` 的对象。这是我们希望线程在开始运行时能够访问的数据。

请注意无论是使用 JavaScript 或者其他能够转译为 JavaScript 的语言（例如 TypeScript），文件路径最后的扩展名应该总是 `.js` 或者 `.mjs` 。

在这里我想指出为什么会使用回调函数的方式而不是在 `message` 事件触发时返回能够被 resolve 的 promise 。这是因为线程工作程序能够多次触发 `message` 事件，而不仅仅是一次。

从上面的例子可以看出，线程之间的通信是基于事件机制的，这也就意味着我们设置了许多监听器函数，当线程工作程序触发某个事件时，相应的监听器函数就能被立即调用。

