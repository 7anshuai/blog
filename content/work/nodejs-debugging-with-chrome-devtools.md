# 在 Chrome DevTools 中并行调试 Node.js 和浏览器 JavaScript

- pubdate: 2017-02-09
- tags: node.js, devtools

------

> Programmers make mistakes. For whimsical reasons, programming errors are called **bugs** and the process of tracking them down is called **debugging**.
>
> -- Allen Downey

最近读了 [Think Python](http://greenteapress.com/wp/think-python-2e/)，书中对 Debugging 的描述很有趣：

> 编写代码，特别是 debugging，有时会产生强烈的情绪。如果你在与一个困难的 bug 做斗争，你可能会感动愤怒，沮丧或者尴尬。

> 有证据表明，人们会很自然的把计算机当作人一样做出反应。当它们运转良好，我们认为它们是好队友，而当它们顽固或粗鲁时，我们也会像对待顽固，粗鲁的人一样回应它们。

> 做点准备可能会帮助你处理这些反应。一种方法是将计算机视为具有某些优势（如速度和精度）和特定弱点（例如缺乏同情心和无法掌握大局）的员工。

> 你的工作是做一个好的经理：找到方法来利用优势和缓解弱点。 并找到使用你的情绪来解决问题的方法，而不会让你的反应干扰你的有效工作能力。

> 学习 debug 可能令人沮丧，但它是一个有价值的技能，对于除了编程以外还有许多活动都是有用的。

对于程序员来说最重要的技能是解决问题。善于 Debug 能极大的提高解决问题的效率和成功率。So, 做为 JS 开发者需要好好学习总结下 JavaScript 及 Node.js debug。

## JavaScript Debugger

Web 浏览器附带一个通常被称为“开发者工具”的内置功能，它提供了更好的视角来观察运行在浏览器里的 JavaScript。虽然不是必须的，但当你调试代码错误时，你会发现开发者工具很有用。

[Chrome DevTools](https://developers.google.cn/web/tools/chrome-devtools/) 是 Google Chrome 中内置的一组网络制作和调试工具。 使用 DevTools 来迭代，调试和配置您的网站。具体的如何调试 JavaScript 代码可以参考[官方文档](https://developers.google.cn/web/tools/chrome-devtools/javascript/)。

## Node debuggers

### Debugger

服务器端的 JavaScript 运行时环境 Node.js 包含了一个基于 TCP 协议访问的进程外调试工具和内置的调试客户端。要使用它，使用debug参数启动Node.js，然后是要调试的脚本的路径; 将显示一个提示，指示调试器成功启动：

```
$ node debug myscript.js
< debugger listening on port 5858
connecting... ok
break in /home/indutny/Code/git/indutny/myscript.js:1
  1 x = 5;
  2 setTimeout(() => {
  3   debugger;
debug>
```

Node.js 的调试客户端不是一个全功能调试器，但简单的步骤和检查是可行的。

更多的详情及高级用法可参考[官方文档](https://nodejs.org/dist/latest-v6.x/docs/api/debugger.html)。

### Node Inspector
随着 Node.js 的大热，开源社区里的优秀开发者贡献了大量的 [debugging 工具](https://github.com/sindresorhus/awesome-nodejs#debugging--profiling)，其中的佼佼者便是 [node-inspector](https://github.com/node-inspector/node-inspector)。

Node Inspector 是 Node.js 应用程序的调试器接口，通过它可以使用 Blink（Chrome 浏览器内核）开发工具来进行 debugging。

#### 安装
通过 npm 安装 node-inspector：
```
$ npm install -g node-inspector
```

#### 启动
然后启动程序：
```
$ node-debug app.js
```
其中 `app.js` 是您的 Node 应用程序主要 JavaScript 文件的名称。

#### 调试
`node-debug` 命令将在默认浏览器中加载 Node Inspector。

> ⚠：Node Inspector 仅适用于 Chrome 和 Opera。 如果另一个浏览器是您的默认网络浏览器（例如 Safari 或 Internet Explorer），则必须在其中一个浏览器中重新打开检查器页面。

Node Inspector 的工作方式几乎与 Chrome DevTools 完全相同。

#### issues

[Node Inspector](https://www.npmjs.com/package/node-inspector) 目前最新的 package 版本是0.12.8，发布于10个月前。
而 Node.js 版本更新很快，v6.x 已经有十几个版本更新了。

在好几个月前，我开始使用 Node.js v6.4.0，同样的也安装了 Node Inspector 来进行 Debugging，然后就遇到了：[Throwing exception on simple use case](https://github.com/node-inspector/node-inspector/issues/905)。

在好长一段时间，这个问题没有得到解决，相关讨论，PR也没有得到回应或者 Merge。直到最近，终于合并了一个 Pull Request：[Fix callback call in InjectorClient._findNMInScope](https://github.com/node-inspector/node-inspector/pull/914)。然而 Node Inspector package 还没有发布 patch 更新，所以 `npm install -g node-inspector` 依然会遇到这个问题。

如果需要在 v6.4.0 以上版本使用 Node Inspector，可以直接从 github 下载安装 `npm install -g https://github.com/node-inspector/node-inspector`，或者手动修改 [InjectorClient.js](https://github.com/node-inspector/node-inspector/pull/914/files)。

### Node.js 集成 V8 Inspector
在 Node Inspector issues里看到了很多讨论，有开发者提出了使用 `node --inspect app.js` 来替代 node-inspector。
原来 Node.js v6.3.0+ 已经内置支持了 [V8 Inspector](https://github.com/nodejs/node/pull/6792)。

[Node.js debugger docs](https://nodejs.org/dist/latest-v6.x/docs/api/debugger.html#debugger_v8_inspector_integration_for_node_js) 文档也增加了这部分内容：

> ⚠：这是一个实验特性。

V8 Inspector 集成允许将 Chrome 开发工具通过 [Chrome Debugging Protocol](https://developer.chrome.com/devtools/docs/debugger-protocol) 连接到 Node.js 实例以进行调试和分析。

V8 Inspector 可以通过在启动 Node.js 应用程序时传递 `--inspect` 标志来启用。 也可以提供具有该标志的自定义端口，例如， `--inspect=9222` 将接受端口9222上的 DevTools 连接。

如需在应用的第一行代码添加断点，在 `--inspect` 之外增加 `--debug-brk` 标志。

```bash
$ node --inspect index.js
Debugger listening on port 9229.
Warning: This is an experimental feature and could change at any time.
To start debugging, open the following URL in Chrome:
    chrome-devtools://devtools/remote/serve_file/@60cd6e859b9f557d2312f5bf532f6aec5f284980/inspector.html?experiments=true&v8only=true&ws=localhost:9229/node
```

## Node.js 与浏览器 JavaScript 并行调试

Chrome DevTools 已经进一步发展，打开具有特定网址的单独页面以调试 Node.js 代码的步骤已非必须。

这意味着，今天你可以在同一个 DevTools 窗口中并行调试浏览器 JavaScript 文件和 Node.js，这有着完美的意义。

### 开启 DevTools 实验特性

目前 Chrome 浏览器 JavaScript 和 Node.js 代码的并行调试是一个实验性的新功能。

要启用它，您必须执行以下操作：
- 打开 [chrome://flags/#enable-devtools-experiments](chrome://flags/#enable-devtools-experiments) URL
- 启用 `Developer Tools experiments` 标志
- 重启 Chrome
- 打开 DevTools 设置 -> Experiments 选项（在重启之后它开始可见）
- 按6次 "SHIFT" 以显示隐藏的实验功能
- 选中 "Node debugging"
- 打开/关闭 DevTools

![Enable DevTools Experiments](http://wx4.sinaimg.cn/mw690/6b4c087fgy1fcoxl9cy10j20zk10sn1m.jpg)

### Debug

开始 debugging，像上面一样以 debug mode 启动 Node.js：
```
node --inspect server.js
```

像平时一样在 Chrome 打开您的页面和 DevTools，选择 Sources -> 按 `ESC` 开启 Console -> 在 debugger 面板中点击 connect 连接 `NodeJS Main Context`：
![Node.js debugging](http://wx3.sinaimg.cn/mw690/6b4c087fgy1fcoxlafw3fj20zk0yyaha.jpg)

如果您的 Node.js 应用有 console.log 或类似输出，您会看到，它们已经出现在 Chrome DevTools console。然后，你可以同时给浏览器和 Node.js 文件设置断点进行 debug。

![Node.js Debugging Gif](http://wx3.sinaimg.cn/mw1024/6b4c087fgy1fcq2n9u9rtg20gn0aenpd.gif)

## 总结

如果您的项目使用 Node.js，现在您可以从一个地方调试和更改所有 JavaScript - Chrome DevTools。

您还可以将 Chrome DevTools 的所有强大功能应用于 Node.js 代码。

## 参考链接
- [Node.js Debugger](https://nodejs.org/dist/latest-v6.x/docs/api/debugger.html)
- [Add V8_inspector support](https://github.com/nodejs/node/pull/6792)
- [Node Inspector](https://github.com/node-inspector/node-inspector)
- [Debugging Node.js with Chrome DevTools](https://medium.com/@paul_irish/debugging-node-js-nightlies-with-chrome-devtools-7c4a1b95ae27#.fuwv7r5ex)
- [Node.js debugging with Chrome DevTools (in parallel with browser JavaScript)](https://blog.hospodarets.com/nodejs-debugging-in-chrome-devtools)
