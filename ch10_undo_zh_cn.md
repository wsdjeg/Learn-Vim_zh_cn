# Undo

对于任何一个现代的软件来说，Undo 都是一个很基本的特性。 Vim 的 undo 系统不仅支持 undo 和 redo 任何修改，而且支持通过修改时间线来修改和还原。 在本章中，你将会学到如何执行 undo 和 redo 文本，浏览 undo 分支，反复 undo, 以及浏览修改时间线。 

# Undo, Redo, and UNDO
对于一个基本的 undo 操作，你可以执行 `u` 或者 `:undo`。
假设你有如下文本：
```
one

```

以及另一个文本：
```
one
two
```

如果你执行 `u`，Vim 会删除 “two”。
Vim 是如何知道应该恢复多少修改呢？ 答案是，Vim每次仅恢复一次修改，这有点类似于 dot 命令的操作（和 dot 命令不同之处在于，命令行命令也会被算作一次修改）。

为了 redo 上一次的修改，执行 `Ctrl-R` 或者 `:redo`。例如，当你执行 undo 来删除 “two” 以后，你可以执行 `Ctrl-R` 来恢复被删除掉的文本。

Vim 也有另一个命令 `U` 可以实现 UNDO 的功能，执行这个命令会 undo 所有最新的修改。
那么，`U` 和 `u` 的区别是什么呢？首先，`U` 会删除 *所有的* 最新的修改，而 `u` 一次仅删除一次修改。 其次，执行`u` 不会被算作一次修改，而执行 `U` 则会被算作一次修改。

让我们会的之前的例子：

```
one
two
```

修改第二行的内容为 “three” (`ciwthree<esc>`):

```
one
three
```

再次修改第二行的例子为 “four” (`ciwfour<esc>`):

```
one
four
```

此时，如果你按下 `u`，你会看到 “three”。如果你再次按下 `u`，你会看到 “two”。然而，在第二行任为 “four” 的时候，如果你按下 `U`，你会看到 

```
one

```

执行 `U` 会跳过中间所有修改，直接恢复到文件最初的状态（第二行为空）。另外，由于 UNO 实际上是执行了一个新的修改，因此你可以 UNDO 执行过的 UNDO。 执行 `U` 后 再次执行 `U` 会 undo 自己。你可以连续执行 `U`，那么你将看到第二行的文本不停的出现和消失。

对我个人而言，并几乎不会使用 `U`，因为很难记住文本最初的样子。如果非说在什么场景下使用 `U`，那就是在我意外地执行了 `Shift-u`。

Vim 可以通过变量 `undolevels` 来选择最多可执行 undo 的次数。你可以通过 `:echo &undolevels` 来查看当前的配置。我一般设置为 1000。如果你也想设置为 1000 的话，你可以执行 `:set undolevels=1000`。不用担心，你可以设置它为任何一个你想设置的值。

# Breaking the Blocks
在上文中我提到，`u` 每次恢复一次修改，类似于 dot 命令。在每次进入 插入模式和推出插入模式之间的任何修改都被定义为一次修改。

如果你执行 `ione two three<esc>` 之后，按下 `u`，Vim 会同时删除 “one two three”，因为这是一笔修改。如果你每次只输入较短的文本，那么这是可接受的，可假设你在一次插入模式中输入了大量的文本并且退出了插入模式，可很快你意识到这中间有部分错误。此时，如果你按下 `u`，你会丢失上一次输入的所有内容。 假设你按下 `u` 只删除你上一次输入的一部分文本岂不是更好。

幸运的是，你可以拆分它。当你在插入模式时，按下 `Ctrl-G u` 会生成一个断点。例如，如果你执行 `ione <Ctrl-G u>two <Ctrl-G u>three<esc>`，之后你按下`u`，你仅会失去文本 “three”，再次执行 `u`，会删除 “two”。当你想要输入一长段内容时，应该有选择性的执行断点插入操作。在每一句话的末尾，两个段落的中间，或者每一行代码的结束插入断点是一个很好的选择，这可以帮助你从错误中恢复出来。

在执行删除操作后插入断点也非常有用，例如通过 `Ctrl-W` 删除光标前的单词，以及 `Ctrl-U`删除光标前的所有文本。一个朋友建议我使用如下的映射：
```
inoremap <c-u> <c-g>u<c-u>
inoremap <c-w> <c-g>u<c-w>
```
通过上述命令，你可以很轻松地恢复被删除的文本。

# Undo Tree

Vim 将每一次修改存储在一个 undo 树中。如果你打开一个空白文件:

```

```

插入一段话：

```
one

```

插入一段话：

```
one
two
```

undo:

```
one

```

插入一段不同的话：

```
one
three
```

再次 undo
```
one

```

插入另一段话：
```
one
four
```


现在如果你执行 undo:
```
one

```

如果你再次执行 undo 操作：

```

```
文本 “one” 也会丢失。对于大部分编辑器来说，找回文本 “two” 和 “three” 都是不可能的事情，但是对于 Vim 来说却不是这样。执行 `g+`，你会得到：

```
one

```

再次执行 `g+` ，你将会看到一位老朋友:
```
one
two
```

让我们继续执行 `g+`:
```
one
three
```

再一次执行 `g+` :
```
one
four
```

在 Vim 中，你每一次执行 `u` 去做一次修改时，Vim都会通过创建一个 “undo 分支”来保存下之前的文本内容。在本例中，你先后输入了 “two”, 执行 `u`，输入“three”，执行 `u`，然后输入 “three”，此时，undo 树已经包含了至少两个叶子节点，主节点包含文本 “three”（最新），分支节点包含文本 “two”。假如你执行了另一次 undo 操作并且输入了 “four”，那么此时会生成三个节点，一个主节点包含文本 “four”, 以及另外两个节点分别存储了 “three” 和 “two”。

为了在几个不同的节点状态间进行切换，你可以执行 `g+` 去获取一个较新的状态，以及执行 `g-` 去获取一个教旧的状态。 `u`， `Ctrl-R`， `g+`， 和 `g-` 之间的区别是，`u` and `Ctrl-R` 只可以在 *main* 节点之间进行切换，而`g+` 和 `g-` 可以在 *所有* 节点之间进行切换。

Undo 树并不可以很轻松地可视化。我发现一个插件 [vim-mundo](https://github.com/simnalamburt/vim-mundo) 对于理解 undo 树很有帮助。花点时间去与它玩耍吧。 

# Persistent Undo
当你通过 Vim 打开一个文件，并且立即按下 `u`，Vim 很可能会显示 “*Already at oldest change*” 的警告。 Vim 可以通过 `:wundo` 保持一份你的 undo 历史记录。

创建一个文件 `mynumbers.txt`. 输入:

```
one
```

插入另一行文件 (确保你要么退出并重新进入插入模式，要么创建了断点):

```
one
two
```

插入新的一行:
```
one
two
three
```

现在，创建你的 undo file。 语法为 `:wundo myundofile`。 如果你需要覆盖一个已存在的文件，在 `wundo` 之后添加 `!`.
```
:wundo! mynumbers.undo
```

退出 Vim。

此时，在目录下，应该有`mynumbers.txt` 和 `mynumbers.undo` 两个文件。再次打开 `mynumbers.txt` 文件并且按下 `u`，这是没有响应的。因为自打开文件后，你没有执行任何的修改。现在，通过执行 `:rundo` 来加载 undo 历史。

```
:rundo mynumbers.undo
```

此时，如果你按下 `u`，Vim 会删除 “three”。再次按下 `u`可以删除 “two”。这就好像你从来没有关闭过 Vim 一样。

如果你想要自动加载 undo 历史文件，你可以通过在你的 `.vimrc` 文件中添加如下代码：
```
set undodir=~/.vim/undo_dir
set undofile
```

我认为将所有的 undo 文件集中保存在一个文件夹中最好，例如在 `~/.vim` 目录下。 `undo_dir` 是随意的。 `set undofile` 告诉 Vim 打开 `undofile` 这个特性，因为该特性默认是关闭的。现在，无论你何时保存，Vim 都会自动创建和保存 undo 的历史记录（在使用`undo_dir`目录前，请确保你已经创建了它）。

# Time Travel
是谁说时间旅行不存在。 Vim 可以通过 `:earlier` 命令将文本恢复为之前的状态。

假如有如下文本:
```
one

```
之后你输入了另一行:
```
one
two
```

如果你输入 “two” 的时间少于10秒，那么你可以通过如下命令恢复到 “two” 还没被输入前的状态:
```
:earlier 10s
```

你可以使用 `:undolist` 去查看之前所做的修改。 `:earlier` 可以加上分钟 (`m`), 小时 (`h`), and 天 (`d`) 作为参数。 

```
:earlier 10s    go to the state 10 seconds before
:earlier 10m    go to the state 10 minutes before
:earlier 10h    go to the state 10 hours before
:earlier 10d    go to the state 10 days before
```

# Learn Undo the Smart Way

`u` 和 `Ctrl-R` 是两个不可缺少的 Vim 参数。请先学会它们。在我的工作流中，我并不使用 UNDO，然而我认为承认它存在是很好的。下一步，学会如何使用`:earlier` 和 `:later`，以及时间参数。在这之后，请花些时间理解 undo 树。 插件 [vim-mundo](https://github.com/simnalamburt/vim-mundo) 对我的帮助很大。单独输入本章中展示的文本，并且查看 undo 树的每一次改变。一旦你掌握它，你看待 undo 系统的眼光一定不同。

在本章之前，你学习了如何在项目内查找任何文本，配合 undo，你可以在时间维度上查找任何一个文本。你现在可以通过位置和写入时间找到任何一个你想找的文本。你已经对 Vim 无所不能了。