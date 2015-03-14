# 基本的命令行编辑技巧

- pubdate: 2015-01-26 22:46:54
- tags: shell

------

GNU Bash shell 提供了 [Command line editing](https://www.gnu.org/software/bash/manual/html_node/Command-Line-Editing.html) 功能，它是由 [Readline library](http://tiswww.case.edu/php/chet/readline/rltop.html) 实现的。Python 交互式命令行和 node.js REPL等程序都实现了类似的命令行编辑功能。Command line editing 支持 Emacs-style 和 Vi-style 的命令风格，默认的是 Emacs-style。

## 命令行编辑

基本上，Unix/Linux 系统默认就支持 Command line editing，所以打开终端，或者在终端运行 Python interactive 或者 Node REPL，都可以马上使用常规的 Emacs 的控制字符集 `Control-*` 了。

- `C-a` （Control-a）移动光标到行首
- `C-e` 移动光标到行尾
- `C-b` 将光标往左移动一个位置
- `C-f` 将光标往右移动一个位置
- `Backspace` 你懂得
- `C-d` 删除光标右边的一个字符
- `C-k` 删除光标右边所有的字符
- `C-y` 拉回最后一次删除的字符
- `C-_` 撤销最后一次操作

## 历史替换

历史替换（History Subsititution）的工作原理如下。所有已运行的非空命令行，都会保存到历史缓冲区，当你在一个新的提示符后输入时，实际上是在缓冲区底部添加新的一行。
`C-p` 可以往上移动一行，`C-n` 往下移动一行，`C-R` 可以反向搜索， `C-s` 向前搜索。

## 键值绑定

可以在 `~/.inputrc` 文件中增加一些自定义的命令和功能。常见的形式是： `key-name: function-name` 或 `"string": function-name`，还可以通过 `set option-name vale` 设置选项。
一个简单的例子：

    # I prefer vi-style editing:
    set editing-mode vi

    # Edit using a single line:
    set horizontal-scroll-mode On

    # Rebind some keys:
    Meta-h: backward-kill-word
    "\C-u": universal-argument
    "\C-x\C-r": re-read-init-file
