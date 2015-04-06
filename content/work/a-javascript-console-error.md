# 一个 JavaScript 控制台错误

- pubdate: 2014-10-16 16:51:17
- tags: javascript, closure

------

Web 浏览器中的开发者工具都会提供一个控制台（或者也叫命令行）。在调试 JS 时，在控制台中打印变量或者测试代码片段都很方便。某日，在调试 JS 代码中，在控制台打印一个变量却出现`ReferenceError: varialbe is not defined`错误，令我感到迷惑不解。场景大致如下：

```javascript
var getSth = function (key) {
    $.ajax({
        url: '/echo/json',
        type: 'GET',
        data: {key: key},
        dataType: 'json'
    }).done(function (data) {
        console.log(key)
    })
}
```

AJAX请求成功执行匿名回调函数时，`console.log`语句会正常打印`key`值。但将`console.log(key)`改为 `console.log(data)`，通过开发工具在此处设置一个断点，代码执行在断点处时在控制台中打印`key`，则会报错`ReferenceError: key is not defined`。

一开始想当然的认为`key`变量是通过查找作用域链获得，反复调试后发现，如果在匿名回调函数的代码中引用`key`变量的话，则会在当前作用域创建一个闭包，而 JavaScript 是基于[词法作用域](http://zh.wikipedia.org/wiki/%E4%BD%9C%E7%94%A8%E5%9F%9F)，闭包的创建是在词法分析阶段，所以在代码执行时通过控制台动态的引用`key`值会得到一个`ReferenceError`。
