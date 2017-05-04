[//]: <> (# A quick guide to writing scripts using the bash shell)
# [译] 使用 bash shell 编写脚本的快速指南

- pubdate: 2017-05-04
- tags: bash, shell
- gh_issue_id: 2

------

> 原文链接：[A quick guide to writing scripts using the bash shell](http://www.panix.com/~elflord/unix/bash-tute.html)

[//]: <> (## A simple shell script)
## 简单的 shell 脚本

一个简单的 shell 脚本只是一点点按顺序执行的命令列表。通常，一个 shell 脚本应该从如下面的一行开始：
[//]: <> (A shell script is little more than a list of commands that are run in sequence. Conventionally, a shellscript should start with a line such as the following:)

```bash
#!/bin/bash
```

这表示脚本应该在 bash shell 中运行，无论用户选择了哪个交互式 shell。这是非常重要的，因为不同 shell 的语法可能有很大差异。
[//]: <> (This indicates that the script should be run in the bash shell regardless of which interactive shell the user has chosen. This is very important, since the syntax of different shells can vary greatly.)

### 简单的例子
[//]: <> (### A simple example)

这是一个非常简单的 shell 脚本示例。 它只是运行一些简单的命令：
[//]: <> (Here's a very simple example of a shell script. It just runs a few simple commands)
```bash
#!/bin/bash
echo "hello, $USER. I wish to list some files of yours"
echo "listing files in the current directory, $PWD"
ls  # list files
```

首先，请注意第4行的注释。在一个 bash 脚本中，任何一个 `#`（除了第一行的 [shebang](https://zh.wikipedia.org/zh-hans/Shebang) 之外）都被视为注释。 即 shell 解释器会忽略它。而对于人们阅读脚本是有益的。
[//]: <> (Firstly, notice the comment on line 4. In a bash script, anything following a pound sign # (besides the shell name on the first line) is treated as a comment. ie the shell ignores it. It is there for the benifit of people reading the script.)

`$USER` 和 `$PWD` 是*变量*。这些是由 bash shell 本身定义的标准变量，它们不需要在脚本中定义。请注意，当变量名称在双引号内时，变量是*展开的*（`expanded`）。展开（`expand`）是一个非常合适的词：shell 看到字符串 `$USER`，并用变量的值替换它，然后执行命令。
[//]: <> (`$USER` and `$PWD` are *variables*. These are standard variables defined by the bash shell itself, they needn't be defined in the script. Note that the variables are *expanded* when the variable name is inside double quotes. Expanded is a very appropriate word: the shell basically sees the string $USER and replaces it with the variable's value then executes the command.)

下面我们来继续讨论变量...
[//]: <> (We continue the discussion on variables below ...)

## 变量
[//]: <> (## Variables)

任何编程语言都需要变量。如下定义一个变量：
[//]: <> (Any programming language needs variables. You define a variable as follows:)

```bash
X="hello"
```

然后引用它：
[//]: <> (and refer to it as follows:)

```bash
$X
```

更具体地说，`$X` 用于表示变量 `X` 的值。 一些要注意的语义：
[//]: <> (More specifically, `$X` is used to denote the value of the variable `X`. Some things to take note of regarding semantics:)
[//]: <> (- bash gets unhappy if you leave a space on either side of the `=` sign. For example, the following gives an error message:)
- 如果你在 `=` 标志的两边留下空格，bash就会变得不快乐。 例如，以下内容导致了一个错误：
```bash
X = hello
```

[//]: <> (- while I have quotes in my example, they are not always necessary. where you need quotes is when your variable names include spaces. For example,)
- 虽然在我的例子中有引号，但并不总是必需的。 当变量的值包含空格时需要引号。 例如：
```bash
X=hello world # error
X="hello world" # OK
```

这是因为 shell 本质上将命令行看作一堆由空格分隔的命令和命令参数。 `foo=baris` 被认为是一个命令。 `foo = bar`的问题是 shell 看到由空格分隔的单词 `foo`，并把它解释为一个命令。 同样，命令 `X=hello world` 的问题是 shell 将 `X=hello` 解释为一个命令，而 `world` 这个词没有任何意义（因为赋值命令不能携带参数）。

[//]: <> (This is because the shell essentially sees the command line as a pile of commands and command arguments seperated by spaces. `foo=baris` considered a command. The problem with `foo = bar` is the shell sees the word `foo` seperated by spaces and interprets it as a command. Likewise, the problem with the command `X=hello world` is that the shell interprets `X=hello` as a command, and the word "world" does not make any sense (since the assignment command doesn't take arguments).)

### 单引号与双引号
[//]: <> (### Single Quotes versus double quotes)

基本上，变量名称只在双引号内展开，单引号里不展开。如果不需要引用变量，单引号很好用，因为结果更可预测。
[//]: <> (Basically, variable names are exapnded within double quotes, but not single quotes. If you do not need to refer to variables, single quotes are good to use as the results are more predictable.)

一个例子：

```bash
#!/bin/bash
echo -n '$USER=' # -n option stops echo from breaking the line
echo "$USER"
echo "\$USER=$USER"  # this does the same thing as the first two lines
```
输出看起来像这样（假设你的用户名是 elflord）:
[//]: <> (The output looks like this (assuming your username is elflord))

```bash
$USER=elflord

$USER=elflord
```

双引号更灵活，但是可预测性较低。如果可以在两者之间选择的话，使用单引号。
[//]: <> (so the double quotes still have a work around. Double quotes are more flexible, but less predictable. Given the choice between single quotes and double quotes, use single quotes.)

### 使用引号括起变量
[//]: <> (### Using Quotes to enclose your variables)

有时，使用双引号保护变量名是个好主意。 如果您的变量值包含空格或是空字符串，则这是很重要的。 一个例子如下：
[//]: <> (Sometimes, it is a good idea to protect variable names in double quotes. This is usually the most important if your variables value either (a) contains spaces or (b) is the empty string. An example is as follows:)

```bash
#!/bin/bash
X=""
if [ -n $X ]; then  # -n tests to see if the argument is non empty
  echo "the variable X is not the empty string"
fi
```

这段脚本会输出：`the variable X is not the empty string`。
因为 shell 将 `$X` 展开为空字符串。 表达式 `[ -n ]` 返回 `true`（因为它没有提供参数）。 一个更好的脚本将是：

[//]: <> (This script will give the following output: the variable X is not the empty string)
[//]: <> (Why ? because the shell expands $X to the empty string. The expression [ -n ] returns true (since it is not provided with an argument). A better script would have been:)

```bash
#!/bin/bash
X=""
if [ -n "$X" ]; then  # -n tests to see if the argument is non empty
  echo "the variable X is not the empty string"
fi
```

在这个例子中，表达式展开为 `[ -n "" ]`，返回 `false`。因为用双引号括起来的字符串显然是空的。

[//]: <> (In this example, the expression expands to [ -n "" ] which returns false, since the string enclosed in inverted commas is clearly empty.)

### 变量展开实战
[//]: <> (### Variable Expansion in action)

只是为了说服你，shell 真的像我之前提到的那样在 “展开” 变量，这里是一个例子：

[//]: <> (Just to convince you that the shell really does "expand" variables in the sense I mentioned before, here is an example:)

```bash
#!/bin/bash
LS="ls"
LS_FLAGS="-al"

$LS $LS_FLAGS $HOME
```

这看起来有点神秘。 最后一行会发生什么，它实际上是执行命令
`ls -al /home/elflord`（假设 `/home/elflord` 是你的主目录）。 也就是说，shell 只是用它们的值替换变量，然后执行命令。
[//]: <> (This looks a little enigmatic. What happens with the last line is that it actually executes the command `ls -al /home/elflord` (assuming that /home/elflord is your home directory). That is, the shell simply replaces the variables with their values, and then executes the command.)

### 使用大括号来保护变量
[//]: <> (### Using Braces to Protect Your Variables)

好了，这里有一个潜在的问题。 假设要 `echo` 变量 `X` 的值，紧接着是字母 `abc`。 问题：你怎么做的？ 我们来试一试：
[//]: <> (OK. Here's a potential problem situation. Suppose you want to echo the value of the variable X, followed immediately by the letters "abc". Question: how do you do this ? Let's have a try:)

```bash
#!/bin/bash
X=ABC
echo "$Xabc"
```

这样没有得到输出。哪里出了错？答案是，shell 认为我们引用的是未初始化的变量 `Xabc`。处理这个问题的方法是把大括号放在 `X` 上以将其与其他字符分开。以下给出了期望的结果：
[//]: <> (This gives no output. What went wrong ? The answer is that the shell thought that we were asking for the variable Xabc, which is uninitialised. The way to deal with this is to put braces around X to seperate it from the other characters. The following gives the desired result:)

```bash
#!/bin/bash
X=ABC
echo "${X}abc"
```

## 条件语句
[//]: <> (## Conditionals, if/then/elif)

有时需要检查某些条件。一个字符串是否有0个长度？文件 “foo” 是否存在，它是一个符号链接还是一个真实的文件？首先，我们使用 `if` 命令来运行测试。 语法如下：
[//]: <> (Sometimes, it's necessary to check for certain conditions. Does a string have 0 length ? does the file "foo" exist, and is it a symbolic link , or a real file ? Firstly, we use the if command to run a test. The syntax is as follows:)

```
if condition
then
  statement1
  statement2
  ..........
fi
```

有时您可能希望在条件失败时指定备用操作。这是如何做的：
[//]: <> (Sometimes, you may wish to specify an alternate action when the condition fails. Here's how it's done.)

```
if condition
then
  statement1
  statement2
  ..........
else
  statement3
fi
```

或者，如果第一个 `if` 失败，则可以测试另一个条件。 请注意，可以添加任何数量的 `elif`。
[//]: <> (alternatively, it is possible to test for another condition if the first "if" fails. Note that any number of elifs can be added.)

```
if condition1
then
  statement1
  statement2
  ..........
elif condition2
then
  statement3
  statement4
  ........    
elif condition3
then
  statement5
  statement6
  ........    

fi
```

如果相应的条件为真，则 `if/elif` 和下一个 `elif` 或 `fi` 之间的块内的语句将被执行。 实际上，任何命令都可以替代条件，并且当且仅当命令返回退出状态为`0`（换句话说，如果命令退出“成功”），则该块将被执行。 但是，在本文档中，我们只会使用 `test` 或 `[ ]` 来测试条件。
[//]: <> (The statements inside the block between if/elif and the next elif or fi are executed if the corresponding condition is true. Actually, any command can go in place of the conditions, and the block will be executed if and only if the command returns an exit status of 0 (in other words, if the command exits "succesfully" ). However, in the course of this document, we will be only interested in using "test" or "[ ]" to evaluate conditions.)

### 测试命令和操作符
[//]: <> (### The Test Command and Operators)

几乎所有的条件语句使用的命令都是测试命令。 测试返回 `true` 或 `false`（更准确地说，退出 `0` 或非零状态），这取决于测试是通过还是失败。 它大概如下工作：
[//]: <> (The command used in conditionals nearly all the time is the test command. Test returns true or false (more accurately, exits with 0 or non zero status) depending respectively on whether the test is passed or failed. It works like this:)

```
test operand1 operator operand2
```

对于某些测试，只需要一个操作数（operand2）测试命令通常以下列形式缩写：
[//]: <> (for some tests, there need be only one operand (operand2) The test command is typically abbreviated in this form:)

```
[ operand1 operator operand2 ]
```

让讨论回到现实，我们举几个例子：
[//]: <> (To bring this discussion back down to earth, we give a few examples:)

```bash
#!/bin/bash
X=3
Y=4
empty_string=""
if [ $X -lt $Y ]  # is $X less than $Y ?
then
  echo "\$X=${X}, which is smaller than \$Y=${Y}"
fi

if [ -n "$empty_string" ]; then
  echo "empty string is non_empty"
fi

if [ -e "${HOME}/.fvwmrc" ]; then       # test to see if ~/.fvwmrc exists
  echo "you have a .fvwmrc file"
  if [ -L "${HOME}/.fvwmrc" ]; then     # is it a symlink ?  
    echo "it's a symbolic link
  elif [ -f "${HOME}/.fvwmrc" ]; then   # is it a regular file ?
    echo "it's a regular file"
  fi
else
  echo "you have no .fvwmrc file"
fi
```

### 值得注意的一些陷阱
[//]: <> (### Some pitfalls to be wary of)

测试命令需要以“operand1 <space> operator <space> operand2”或“operator <space> operand2”的形式，换句话说，您真的需要这些空格，因为 shell 认为第一个不包含空格的块是运算符（如果以 `-` 开头）或操作数。例如：
[//]: <> (The test command needs to be in the form "operand1<space>operator<space>operand2" or operator<space>operand2 , in other words you really need these spaces, since the shell considers the first block containing no spaces to be either an operator (if it begins with a '-') or an operand (if it doesn't). So for example; this)

```bash
if [ 1=2 ]; then
  echo "hello"
fi
```

以上会给出准确的 “错误” 输出（即 `echo "hello"`，因为 shell 看到一个操作数，但没有操作符）。

另一个潜在的陷阱来自于不保护引号中的变量。 我们已经给出了一个例子，说明为什么你必须用引号括起你想要使用在 `-n` 测试中的操作数。而且，在大部分时候都有很多足够好的理由来使用引号。当您在测试中展开变量时，无法执行此操作可能会导致非常严重的错误。 以下是一个例子：
[//]: <> (gives exactly the "wrong" output (ie it echos "hello", since it sees an operand but no operator.))
[//]: <> (Another potential trap comes from not protecting variables in quotes. We have already given an example as to why you must wrap anything you wish to use for a -n test with quotes. However, there are a lot of good reasons for using quotes all the time, or almost all of the time. Failing to do this when you have variables expanded inside tests can result in very wierd bugs. Here's an example: For example,)

```bash
#!/bin/bash
X="-n"
Y=""
if [ $X = $Y ] ; then
  echo "X=Y"
fi
```

这将导致错误输出，因为 shell 将我们的表达式展开为`[ -n = “=” ]`，字符串 “=” 具有非零长度。
[//]: <> (This will give misleading output since the shell expands our expression to `[ -n = ]` and the string "=" has non zero length.)

### 测试操作符的简要总结
[//]: <> (### A brief summary of test operators)

以下是测试运算符的快速列表。 这不是全面的，但它可能是所有你需要记住的（如果你需要任何其他的，你可以随时检查 bash 手册页...）
[//]: <> (Here's a quick list of test operators. It's by no means comprehensive, but its likely to be all you'll need to remember (if you need anything else, you can always check the bash manpage ... ))

operator | produces true if... | number of operands
--- | --- | ---
-n | operand non zero length | 1
-z | operand has zero length | 1
-d | there exists a directory whose name is operand | 1
-f | there exists a file whose name is operand | 1
-eq | the operands are integers and they are equal | 2
-neq | the opposite of -eq | 2
= | the operands are equal (as strings) | 2
!= | opposite of = | 2
-lt | operand1 is strictly less than operand2 (both operands should be integers) | 2
-gt | operand1 is strictly greater than operand2 (both operands should be integers) | 2
-ge | operand1 is greater than or equal to operand2 (both operands should be integers) | 2
-le | operand1 is less than or equal to operand2 (both operands should be integers) | 2

## 循环
[//]: <> (## Loops)

循环是使得人们可以重复一个过程或对几个不同的项目执行相同的过程的结构。 在bash中有以下种类的循环可用：
[//]: <> (Loops are constructions that enable one to reiterate a procedure or perform the same procedure on several different items. There are the following kinds of loops available in bash)

- for 循环
- while 循环

### for 循环
[//]: <> (### For loops)

for 循环的语法最适合通过示例描述：
[//]: <> (The syntax for the for loops is best demonstrated by example.)

```bash
#!/bin/bash
for X in red green blue
do
  echo $X
done
```

for 循环遍历空白分隔的项。 请注意，如果某些项具有嵌入空白，则需要使用引号保护它们。 以下是一个例子：
[//]: <> (The for loop iterates the loop over the space seperated items. Note that if some of the items have embedded spaces, you need to protect them with quotes. Here's an example:)

```bash
#!/bin/bash
colour1="red"
colour2="light blue"
colour3="dark green"
for X in "$colour1" $colour2" $colour3"
do
  echo $X
done
```
你能猜测如果我们在 for 语句中省略引号，会发生什么？ 这表明变量名应该用引号保护，除非你确定它们不包含任何空格。
[//]: <> (Can you guess what would happen if we left out the quotes in the for statement ? This indicates that variable names should be protected with quotes unless you are pretty sure that they do not contain any spaces.)

#### for 循环中的 glob
[//]: <> (#### Globbing in for loops)

shell 将包含 `*` 的字符串扩展为“匹配”的所有文件名。当且仅当与匹配字符串相同时，文件名匹配，用任意字符串替换星号 `*` 后。 例如，字符 `*` 本身扩展到工作目录中所有文件的空格分隔列表（不包括以点开头的 `.`）。所以：
[//]: <> (The shell expands a string containing a * to all filenames that "match". A filename matches if and only if it is identical to the match string after replacing the stars * with arbitrary strings. For example, the character "*" by itself expands to a space seperated list of all files in the working directory (excluding those that start with a dot "." ) So)
- `echo *` 列出当前目录中的所有文件和目录
- `echo *.jpg` 列出所有 jpeg 文件
- `echo ${HOME}/public_html/*.jpg` 列出您的 public_html 目录中的所有 jpeg 文件

正因为如此，这对于对目录中的文件执行操作非常有用，特别是与for循环一起使用。 例如：
[//]: <> (As it happens, this turns out to be very useful for performing operations on the files in a directory, especially used in conjunction with a for loop. For example:)

```bash
#!/bin/bash
for X in *.html
do
    grep -L '<UL>' "$X"
done
```

### while 循环
[//]: <> (### While Loops)

当一个给定的条件为真 while 循环进行迭代。 一个例子：
[//]: <> (While loops iterate "while" a given condition is true. An example of this:)

```bash
#!/bin/bash
X=0
while [ $X -le 20 ]
do
  echo $X
  X=$((X+1))
done
```

这提出了一个自然的问题：为什么 bash 不允许 C 语言式的 for 循环 `for（X = 1，X <10; X ++）` ？
[//]: <> (This raises a natural question: why doesn't bash allow the C like for loops `for (X=1,X<10; X++)`)

事实上，for 循环不被鼓励使用，因为：bash 是一种解释性语言，而且它的循环是一个相当缓慢的事情。 因此，不鼓励重复迭代。
[//]: <> (As it happens, this is discouraged for a reason: bash is an interpreted language, and a rather slow one for that matter. For this reason, heavy iteration is discouraged.)

## 命令替换
[//]: <> (## Command Substitution)

命令替换是 bash shell 非常方便的功能。 它使您可以获取命令的输出，并将其视为在命令行中写入。 例如，如果要将变量 `X` 设置为命令的输出，则通过命令替换来执行此操作。
[//]: <> (Command Substitution is a very handy feature of the bash shell. It enables you to take the output of a command and treat it as though it was written on the command line. For example, if you want to set the variable X to the output of a command, the way you do this is via command substitution.)

有两种形式的命令替换：括号展开和反向展开。
[//]: <> (There are two means of command substitution: brace expansion and backtick expansion.)

括号扩展工作如下：`$(commands)` 展开到命令的输出，允许嵌套。因此命令可以包括括号扩展：
[//]: <> (Brace expansion workls as follows: $(commands) expands to the output of commands This permits nesting, so commands can include brace expansions)

反向扩展将 `commands` 展开到命令的输出：
[//]: <> (Backtick expansion expands `commands` to the output of commands)

给出一个例子：
[//]: <> (An example is given:)

```bash
#!/bin/bash
files="$(ls)"
web_files=`ls public_html`
echo "$files"      # we need the quotes to preserve embedded newlines in $files
echo "$web_files"  # we need the quotes to preserve newlines
X=`expr 3 \* 2 + 4` # expr evaluate arithmatic expressions. man expr for details.
echo "$X"
```

`$()` 替代方法的优点是几乎不言而喻：嵌套很容易。 大部分的 bourne shell 可以支持（或 POSIX shell）。但是，反向替换稍微可读性更好，甚至最基本的 shell 也支持（任何 `#!/bin/sh` 都很好）。
[//]: <> (The advantage of the $() substitution method is almost self evident: it is very easy to nest. It is supported by most of the bourne shell varients (the POSIX shell or better is OK). However, the backtick substitution is slightly more readable, and is supported by even the most basic shells (any #!/bin/sh version is just fine))

请注意，如果字符串在上述 `echo` 语句中没有引号保护，换行符将被输出中的空格替换。
[//]: <> (Note that if strings are not quote-protected in the above echo statement, new lines are replaced by spaces in the output.)
