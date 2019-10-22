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

想要使用线程工作程序，我们就得引入 `worker_threads` 模块。首先让我们来创建一个能够帮助我们生成线程工作程序的函数，然后再来讲讲它的内部实现。

```ts
type WorkerCallback = (err: any, result?: any) => any

export function runWorker(path: string, cb: WorkerCallback, workerData: object | null = null) {
 const worker = new Worker(path, { workerData })
 
 worker.on('message', cb.bind(null, null))
  
 worker.on('error', cb)
  
 worker.on('exit', (exitCode) => {
   if (exitCode === 0) {
     return null
   }
   return cb(new Error(`Worker has stopped with code ${exitCode}`))
 })
  
 return worker
}
```

为了创建线程工作程序，我们就得生成 `Worker` 类的实例。该函数的第一个参数是包含线程工作程序代码的文件路径，在第二个参数中我们提供了一个包含属性名为 `workerData` 的对象。这是我们希望线程在开始运行时能够访问的数据。

请注意无论是使用 JavaScript 或者其他能够转译为 JavaScript 的语言（例如 TypeScript），文件路径最后的扩展名应该总是 `.js` 或者 `.mjs` 。

在这里我想指出为什么会使用回调函数的方式而不是在 `message` 事件触发时返回能够被 resolve 的 promise 。这是因为线程工作程序能够多次触发 `message` 事件，而不仅仅是一次。

从上面的例子可以看出，线程之间的通信是基于事件机制的，这也就意味着我们设置了许多监听器函数，当线程工作程序触发某个事件时，相应的监听器函数就会被立即调用。

下面是一些最常见的事件：

```js
worker.on('error', (error) => {})
```

每当线程工作程序中出现未捕获异常时，`error` 事件就会被触发。然后线程工作程序就会被终止，错误信息则作为回调函数的第一个参数传入。

```js
worker.on('exit', (exitCode) => {})
```

当线程工作程序退出时会触发 `exit` 事件。如果在线程工作程序中出现 `process.exit()` ，回调函数中就会出现相应的 `exitCode` 。如果是通过调用 `worker.terminate()` 终止的，`exitCode` 则会为 1 。

```js
worker.on('online', () => {})
```

当线程工作程序解析完 JavaScript 代码并开始执行时，`online` 事件就会被触发。虽然这个事件不常用，但它可以在特定场景下提供信息。

```js
worker.on('message', (data) => {})
```

当线程工作程序向父线程发送数据时 `message` 事件就会被触发。

现在让我们来看一下线程之间是如何共享数据的吧。

在线程之间交换数据
-----

我们通过 `port.postMessage()` 方法将数据从一个线程发送至另一个线程。该方法有如下特性：

```js
port.postMessage(data[, transferList])
```

port 对象既可以是 `parentPort` 也可以是 `MessagePort` 类的实例  — 通常我们使用后者。

#### data 参数

该方法的第一个参数名为 `data` — 它是一个能够拷贝到其他线程的对象。它包含着拷贝算法支持的所有内容。

`data` 通过[结构化克隆算法](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm)拷贝，Mozilla 上是这样解释的：

> 它通过递归输入对象来构建克隆，同时保持先前访问过的引用的映射，以避免无限遍历循环。

该算法并不会拷贝函数、错误、属性描述符或者原型链。同时还应该注意，这种拷贝对象的方式与 JSON 不同，因为它可以包含循环引用和类型数组，而 JSON 则不能。

由于支持类型数组的拷贝，该算法使得在线程之间共享内存成为可能。

#### 在线程之间共享内存

人们可能会说像 `cluster` 或者 `child_process` 这样的模块在很早之前就能使用线程。但这句话只说对了一半。

`cluster` 模块能够在主进程下创建多个节点实例，并通过它们来控制路由请求。集群应用程序能够有效地增加服务器吞吐量。然而，我们并不能通过 `cluster` 模块生成一个单独的线程。

人们倾向于使用 PM2 这样的工具来集群他们的应用程序，而不是在代码中手动实现这一过程。如果你感兴趣的话，可以看看之前我写的关于 `cluster` 模块的[文章](https://medium.freecodecamp.org/how-to-add-socket-io-to-multi-threaded-node-js-df404b424276)。

`child_process` 模块能够生成任何的可执行文件，不管它是不是 JavaScript 。它与 `worker_threads` 非常相似，但却缺少了几个十分重要的特性。

具体来讲，线程工作程序更加的轻量，并且它与父线程共享相同的进程 ID 。它们同时还能与其父线程共享内存，这使得它们可以避免序列化大的数据，因此可以更有效地来回发送数据。

让我们来看一个如何在线程之间共享内存的例子。为了能够共享内存，`port.postMessage()` 方法中的 `data` 参数必须是 `ArrayBuffer` 或者 `SharedArrayBuffer` 类的实例对象。

下面是一个线程工作程序与其父线程共享内存的例子：

```js
import { parentPort } from 'worker_threads'

parentPort.on('message', () => {
 const numberOfElements = 100
 const sharedBuffer = new SharedArrayBuffer(Int32Array.BYTES_PER_ELEMENT * numberOfElements)
 const arr = new Int32Array(sharedBuffer)
 
 for (let i = 0; i < numberOfElements; i += 1) {
   arr[i] = Math.round(Math.random() * 30)
 }
 
 parentPort.postMessage({ arr })
})
```

首先，我们用包含 100 个 32 位整数所需的内存创建了一个 `SharedArrayBuffer` 类的实例对象。然后我们又创建了 `Int32Array` 类的实例对象，并将 buffer 对象作为参数传入。最后我们用随机数字将该数组填满并发送给父线程。

在父线程中，我们有着这样的逻辑代码：

```js
import path from 'path'
import { runWorker } from '../run-worker'

const worker = runWorker(path.join(__dirname, 'worker.js'), (err, { arr }) => {
 if (err) {
   return null
 }
 arr[0] = 5
})

worker.postMessage({})
```

通过将 `arr[0]` 改为 `5` ，我们其实在两个线程上都对它进行了修改。

当然，共享内存也会为我们带来在一个线程中更改值而另一个线程中的值也会被更改的风险。但同时我们也获得了一个很好的特性：值不需要被序列化就能够在另一个线程中使用，这极大地提高了效率。我们只需要记住正确地管理对数据的引用，以便垃圾收集机制对数据进行回收。

共享整数数组虽然可以，但我们真正感兴趣的是共享对象 — 存储信息最常用的方式。不幸的是，我们并没有 `SharedObjectBuffer` 类或者相似的机制，但是我们可以自己创建一个[类似的结构](https://stackoverflow.com/questions/51053222/nodejs-worker-threads-shared-object-store)。

#### transferList 参数

`transferList` 参数只允许包含 `ArrayBuffer` 和 `MessagePort` 类。一旦它们被转移到另一个线程，原来的线程就不能够使用它们了。因为内存被移动到了另一个线程，所以在原来的线程中就不可用了。

目前，我们还不能通过 `transferList` 传送网络套接字（但我们可以通过 `child_process` 模块来实现）。

#### 为线程间通信创建通道

线程之间通过 port 进行通信。第一种情况是默认也是更简单的通信方式。在线程工作程序的文件代码中，我们引入了 `worker_threads` 模块并使用该对象的 `.postMessage()` 方法将消息传送给父线程。

下面是一个例子：

```js
import { parentPort } from 'worker_threads'

const data = {
 // ...
}

parentPort.postMessage(data)
```

`parentPort` 是 Node.js 在后台为我们创建的 `MessagePort` 类的实例，用于支持与父线程的通信。通过这种方式，我们可以使用 `parentPort` 和 `woker` 对象在线程之间进行通信。

第二种在线程间通信的方式其实是我们自己创建一个 `MessageChannel` 类的实例，然后将它发送给线程工作程序。下面是一个如何创建 `MessageChannel` 类的实例对象并在线程工作程序间共享的例子：

```js
import path from 'path'
import { Worker, MessageChannel } from 'worker_threads'

const worker = new Worker(path.join(__dirname, 'worker.js'))
const { port1, port2 } = new MessageChannel()

port1.on('message', (message) => {
 console.log('message from worker:', message)
})

worker.postMessage({ port: port2 }, [port2])
```



