# [译] 为 Node.js 包准备的 Makefile

- pubdate: 2015-04-12
- tags: node.js, shell

-------

> 原文链接：[Makefile recipes for Node.js packages](https://andreypopp.com/posts/2013-05-16-makefile-recipes-for-node-js.html)

当你编写 Node.js 代码时，在构建－测试－发布周期中肯定会有一些事情需要自动化。我使用 `make` 程序来完成这些任务，主要是因为它的简单明了。

要开始使用 `make` 你需要在项目根目录创建一个 `Makefile`。`Makefile` 包含变量和任务声明（下面会给出例子）。要执行一些任务，你只需要在命令行执行 `make <task name>`。简单吧！

以下是我开始一个新的 Node.js 或者 Browserify 项目的通用 `Makefile` 文件。我会一步一步的完成它。

首先是定义一些有用的变量：我在 `src` 保存源码，编译过的代码放在 `lib`（是的，我使用 CoffeeScript，但你可以随意使用你喜欢的语言来定制模板）。

```sh
BIN = ./node_modules/.bin
SRC = $(wildcard src/*.coffee)
LIB = $(SRC:src/%.coffee=lib/%.js)
```

`SRC` 将会包含一个 `src` 目录中的 `.coffee` 文件列表，然后 `LIB` － 一个相对应的 `.js` 文件列表（目前还不存在）。`$(VAR:pattern1=pattern2)` 使用来指定存储在变量中的每一个项的变换。

所以如果我们保存 `src/index.coffee src/mod.coffee` 在文件系统中， `SRC` 会捕获它们然后相应的使 `LIB` 保存 `lib/index.js lib/mod.js` 。

`BIN` 指向 Node 本地可执行模块的安装目录。

## 构建

现在让我们定义第一个任务构建并表明它依赖于存储在 `LIB` 变量的所有文件。

```bash
build: $(LIB)
```

很简单，对不对？`$(LIB)` 只是在 `make` 中间接引用变量的语法。

运行 `make build` 后，程序会尝试确保 `LIB` 中的所有文件都已就位并及时更新。但是我们如何让 `make` 知道怎样在 `SRC` 中获取相应的文件，处理并将所有的这些文件放入 `LIB` 中呢？

接下来的片段定义了一个叫做通配符的规则，它作用于通过给定模式匹配的文件，在当前情况下 － `lib/%.js` － 正是这个模式将会在 `LIB` 变量中进行文件匹配。

```bash
lib/%.js: src/%.coffee
  @mkdir -p $(@D)
  @$(BIN)/coffee -bcp $< > $@
```

这条规则告诉 `make` 那些 `lib/%.js` 文件依赖于相应的 `src/%.coffee` 文件，所以如果当后者发生改变时 `make` 会重新编译生成前者。

它是如何工作的？首先，它创建了一个目标文件的目录（`$(@D)` 表示这个目录，它是 `make` 中的一个魔法变量），然后调用 CoffeeScript 编译相应的 `.coffee` 文件（通过 `$<` 表示）并将结果写入目标文件（通过 `$@` 表示）。

注意 @ 前缀，默认的 `make` 会打印所有执行的命令行，但有 @ 前缀的命令行不会打印。

足够作为一个构建程序了，`make build` 会从 src 目录下将相应的文件重新构建到 lib 目录下。

## 测试

测试任务很简单：

```bash
test: build
  @$(BIN)/mocha -b specs
```

我们只需指明测试依赖构建任务 － 我们要测试最新的代码 － 然后运行选择的测试工具（当前场合是 mocha）。

## 辅助任务

接下来到辅助任务 － `clean` 用来清除所有编译生成的代码：

```bash
clean:
  @rm -f $(LIB)
```

`install` 和 `link` 任务是简单的运行相应的 `npm` 子命令：

```bash
install link:
  @npm $@;
```

注音 `$@` 变量的使用技巧，它是如何传递任务名称作为 `npm` 的子命令。

## 版本

下一个是版本任务。

下面的片段可能看起来有些复杂，但实际上它挺简单的。它定义了一个参数化的宏，用来打补丁，生成次要和主要版本。它被以下的任务所使用。

```bash
define release
  VERSION=`node -pe "require('./package.json').version"` && \
  NEXT_VERSION=`node -pe "require('semver').inc(\"$$VERSION\", '$(1)')"` && \
  node -e "\
    var j = require('./package.json');\
    j.version = \"$$NEXT_VERSION\";\
    var s = JSON.stringify(j, null, 2);\
    require('fs').writeFileSync('./package.json', s);" && \
  git commit -m "release $$NEXT_VERSION" -- package.json && \
  git tag "$$NEXT_VERSION" -m "release $$NEXT_VERSION"
endef
```

简而言之，它使用一个新的递增版本号（版本递增的部分是通过宏参数 `$1` 变量定义的）重写了 `package.json`，然后创建了一个相应的 `git commit` 和 `git tag`。

接下啦只需要通过传递 `patch`，`minor` 或者 `major` 参数调用 `release` 宏来创建相应的任务，这些任务依赖构建和测试任务。这样如果不能通过构建或者测试则不能生成版本。

```bash
release-patch: build test
  @$(call release,patch)

release-minor: build test
  @$(call release,minor)

release-major: build test
  @$(call release,major)
```

最后一点是 `publish` 任务，它用来推送代码到仓库，并发布包到 npm。

```bash
publish:
  git push --tags origin HEAD:master
  npm publish
```

现在发布一个新的次要版本只需要在命令行执行 `make release-minor publish` － `package.json` 中的次要版本会递增，新的 Git 提交和标记会被创建和推送到仓库，最后 Node.js 包也会被发布在 npm。

完整的 `Makefile` 在 [这里](https://gist.github.com/5588256)。
