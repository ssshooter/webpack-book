#Web Workers
# Web Workers

[Web worker]（https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API）允许您将工作推送到JavaScript的主执行线程之外，这使得它们便于进行冗长的计算和后台工作。
[Web workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) allow you to push work outside of main execution thread of JavaScript making them convenient for lengthy computations and background work.

在主线程和工作线程之间移动数据会带来与通信相关的开销。拆分提供了隔离，迫使工作人员只关注逻辑，因为他们无法直接操作用户界面。
Moving data between the main thread and the worker comes with communication-related overhead. The split provides isolation that forces workers to focus on logic only as they cannot manipulate the user interface directly.

工人的想法在更一般的层面上是有价值的。 [parallel-webpack]（https://www.npmjs.com/package/parallel-webpack）使用下面的[worker-farm]（https://www.npmjs.com/package/worker-farm）并行化webpack执行。
The idea of workers is valuable on a more general level. [parallel-webpack](https://www.npmjs.com/package/parallel-webpack) uses [worker-farm](https://www.npmjs.com/package/worker-farm) underneath to parallelize webpack execution.

正如* Build Targets *章节中所讨论的，webpack允许您将应用程序构建为工作程序本身。为了更好地了解Web工作者，您将学习如何使用[worker-loader]（https://www.npmjs.com/package/worker-loader）构建一个小工作者。
As discussed in the *Build Targets* chapter, webpack allows you to build your application as a worker itself. To get the idea of web workers better, you'll learn how to build a small worker using [worker-loader](https://www.npmjs.com/package/worker-loader).

由于[SharedArrayBuffer]等技术（https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer），未来主机和工作人员之间的数据共享可能会变得更加容易）。
T> Sharing data between the host and the worker may become easier in the future thanks to technologies such as [SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer).

##设置工作者加载器
## Setting Up Worker Loader

首先，将* worker-loader *安装到项目中：
To get started, install *worker-loader* to the project:

```bash
npm install worker-loader --save-dev
```

您可以使用内联加载程序定义来最小化演示，而不是将加载程序定义推送到webpack配置。有关替代方案的更多信息，请参阅* Loader Definitions *一章。
Instead of pushing the loader definition to webpack configuration, you can use inline loader definitions to keep the demonstration minimal. See the *Loader Definitions* chapter for more information about the alternatives.

##设置工人
## Setting Up a Worker

工作人员必须做两件事：听取消息并做出回应。在这两个动作之间，它可以执行计算。在这种情况下，您接受文本数据，将其附加到自身，并发送结果：
A worker has to do two things: listen to messages and respond. Between those two actions, it can perform a computation. In this case, you accept text data, append it to itself, and send the result:

** SRC / worker.js **
**src/worker.js**

```javascript
self.onmessage = ({ data: { text } }) => {
  self.postMessage({ text: text + text });
};
```

##设置主机
## Setting Up a Host

主机必须实例化工作者，然后与之通信。除了主机拥有控件之外，这个想法几乎相同：
The host has to instantiate the worker and then communicate with it. The idea is almost the same except the host has the control:

** SRC / component.js **
**src/component.js**

```javascript
import Worker from "worker-loader!./worker";

export default () => {
  const element = document.createElement("h1");
  const worker = new Worker();
  const state = { text: "foo" };

  worker.addEventListener("message", ({ data: { text } }) => {
    state.text = text;
    element.innerHTML = text;
  });

  element.innerHTML = state.text;
  element.onclick = () => worker.postMessage({ text: state.text });

  return element;
};
```

你有这两个设置后，它应该工作。当您单击文本时，它应该在工作程序完成其执行时改变应用程序状态。为了演示工作者的异步性质，您可以尝试在答案中添加延迟并查看会发生什么。
After you have these two set up, it should work. As you click the text, it should mutate the application state as the worker completes its execution. To demonstrate the asynchronous nature of workers, you could try adding delay to the answer and see what happens.

T> [webworkify-webpack]（https://www.npmjs.com/package/webworkify-webpack）是* worker-loader *的替代方案。 API允许您将worker用作常规JavaScript模块，同时避免在示例解决方案中看到“self”要求。 [webpack-worker]（https://www.npmjs.com/package/webpack-worker）是另一个学习的选择。
T> [webworkify-webpack](https://www.npmjs.com/package/webworkify-webpack) is an alternative to *worker-loader*. The API allows you to use the worker as a regular JavaScript module as well given you avoid the `self` requirement visible in the example solution. [webpack-worker](https://www.npmjs.com/package/webpack-worker) is another option to study.

##结论
## Conclusion

需要注意的关键是工作者无法访问DOM。您可以在工作程序中执行计算和查询，但它无法直接操作用户界面。
The critical thing to note is that the worker cannot access the DOM. You can perform computation and queries in a worker, but it cannot manipulate the user interface directly.

回顾一下：
To recap:

* Web worker允许您将工作推出浏览器的主线程。如果性能是一个问题，这种分离是有价值的。
* Web workers allow you to push work out of the main thread of the browser. This separation is valuable especially if performance is an issue.
* Web worker无法操纵DOM。相反，最好将它们用于冗长的计算和请求。
* Web workers cannot manipulate the DOM. Instead, it's best to use them for lengthy computations and requests.
* Web工作人员提供的隔离可用于架构优势。它迫使程序员留在特定的沙箱中。
* The isolation provided by web workers can be used for architectural benefit. It forces the programmers to stay within a specific sandbox.
*与网络工作者沟通带来的开销使他们不那么实用。随着规范的发展，这可能会在未来发生变化。
* Communicating with web workers comes with an overhead that makes them less practical. As the specification evolves, this can change in the future.

您将在下一章了解国际化。
You'll learn about internationalization in the next chapter.

