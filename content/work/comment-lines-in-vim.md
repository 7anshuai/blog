# 在 Vim 中多行注释

- pubdate: 2014-06-17 21:47:14
- tags: vim

------

[What's a quick way to comment/uncomment lines in vim?](http://stackoverflow.com/questions/1676632/whats-a-quick-way-to-comment-uncomment-lines-in-vim)

Stackoverflow 上的一个关于 Vim comments 的问题有很多不错的答案，记录其中一个简单基础的方法。

- 首先，将光标移动到想要注释的代码块第一行
- 然后 `Ctrl + V` （`Ctrl + Q` for Gvim）进入 Visual block mode
- 移动光标到要注释的代码末行
- `Shift + i`（大写 I）
- 添加行注释 `//` 或 `#`
- 最后按下 `Esc`

Give it a second to work.
