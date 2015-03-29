# 在 Node.js 中使用 Asset Pipeline

- pubdate: 2015-03-16 22:03:07
- tags: node.js

------

> There are only two hard things in Computer Science: cache invalidation and naming things.
>
> -- Phil Karlton

计算机科学只有两个难题：缓存失效和变量命名。

*Coding* 中，这两道难题确实无处不在。难题之一缓存失效，Web 中的资源缓存涉及到服务器和浏览器两端的各种缓存机制（详情可阅读腾讯 AlloyTeam 的博客 [Web 缓存机制系列](http://alloyteam.com/2012/03/web-cache-1-web-cache-overview/)）。当 Web 应用进行版本更新时，需要发布新的资源文件并更新缓存。那怎么让 Web 浏览器中原有的缓存失效，加载新的资源文件并缓存到客户端的计算机上呢？常见的两种做法是：

- 传统手工作业
- 基于日期的请求字符串

这两种方法都有明显的缺点，手动控制文件版本的做法通常是这样的：

```html
<!-- version 0.0.1 -->
<script src="http://cdn.example.com/static/0.0.1/js/app.js"></script>

<!-- version 0.0.2 -->
<script src="http://cdn.example.com/static/0.0.2/js/app.js"></script>
```

基于日期的请求字符串：

```html
<!-- version 0.0.1 -->
<script src="http://cdn.example.com/static/js/app.js?1427594349480"></script>

<!-- version 0.0.2 -->
<script src="http://cdn.example.com/static/js/app.js?1427594349481"></script>
```

这两种方法都有明显的缺点,传统手工作业是繁琐低效的，而基于日期的请求字符串的缓存并不可靠。在查看过 [GitHub](https://github.com/) 的网页源码后，发现他们组织程序的静态资源方式有所不同：

```html
<script src="https://assets-cdn.github.com/assets/github-d869f6edeea2dbd9c7c3595e2f31cf8a1530bd36eaa84707461f65c5ee848853.js"></script>
```

文件名后面加上了一串字符串（基于MD5生成），顿觉莫名的高大上啊。偶尔在 [Ruby China](https://ruby-china.org/) 闲逛得知，这是 Rails 3.1 版本开始引入的静态资源管理方式 [Asset Pipeline](http://guides.ruby-china.org/asset_pipeline.html)。

## Asset Pipeline 是什么？

Rails 指南中提到，Asset Pipeline 提供了一个框架，用于连接、压缩 JavaScript 和 CSS 文件。还允许使用其他语言和预处理器编写 JavaScript 和 CSS，例如 CoffeeScript、Sass 和 ERB。它提供了三个主要功能：

- 连接合并 JavaScript 和 CSS 文件，减少页面的 HTTP 请求。
- 压缩 JavaScript 和 CSS 文件（减重瘦身上前线）
- 高级语言及预处理器支持，允许使用高级语言编写静态资源，再使用预处理器转换成真正的静态资源。默认支持用来编写 CSS 的 Sass，用来编写 JavaScript 的 CoffeeScript。

在生产环境中，Rails 通过 Asset Pipeline 技术在文件名后加上 MD5 指纹，以便浏览器缓存，指纹变了缓存就会过期。修改文件的内容后，指纹会自动变化。

## MD5 指纹

Rails 指南中详细解释了指纹。指纹可以根据文件内容生成文件名，文件内容变化后，文件名也会改变。对于静态内容，或者很少改动的内容，在不同的服务器之间，不同的部署日期之间，使用指纹可以区别文件的两个版本内容是否一样。

如果文件名基于内容而定，而且文件名是唯一的，HTTP 报头会建议在所有可能的地方（CDN，ISP，网络设备，网页浏览器）存储一份该文件的副本。修改文件内容后，指纹会发生变化，因此远程客户端会重新请求文件。这种技术叫做“缓存爆裂”（cache busting）。

## connect-assets

不同语言不同框架都有类似 Rails Asset Pipeline 的实现，[connect assets](https://github.com/adunkman/connect-assets) 是为 Node.js 打造的 asset pipeline。它也实现了以上所述的三个主要功能：合并，压缩 JavaScript/CSS 文件，高级语言预处理。在 Node.js 中也可以给静态资源添加指纹，使用更有效的缓存技术。

使用方法也很简单，第一步在项目中安装 connect asset：

```bash
npm install connect-asset
```

第二步，在 Express 应用中添加配置代码：

```javascript
app.use(require('connect-assets')());
```

最后，在项目中创建一个 `assets` 文件夹，并分别将 JavaScript 和 CSS 文件放入 `/assets/js` 和 `/assets/css`。

Node.js 应用就可以使用最基本的 asset pipeline 功能了，更多高级功能请参考官方文档。一个使用 connect assets 的 Node.js App [Node starter](http://node.tanshuai.me/)，fork from [Hackathon Starter](https://github.com/sahat/hackathon-starter)。

扩展阅读：

- [Optimize caching](http://code.google.com/speed/page-speed/docs/caching.html)
- [Revving Filenames: dont't use querystring](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/)

周末福利:

豆瓣女神，独立音乐人，北大美女才女 － 程璧的官方主页 [Anna's Patio](http://www.annapatio.com/) 是基于 Ruby on Rails，真心很赞啊！

