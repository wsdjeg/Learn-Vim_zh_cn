# 第20章 视图、会话和 Viminfo

当您做了一段时间的项目后，您可能会发现这个项目逐渐形了成自己的设置、折叠、缓冲区、布局等，就像住了一段时间公寓后，精心装饰了它一样。问题是，关闭 Vim 后，所有的这些更改都会丢失。如果能保留这些更改，等到下次打开 Vim 时，一切恢复如初，岂不美哉？

本章中，您将学习如何使用 视图、会话 和 Viminfo 来保存项目的“快照”。

## 视图

视图是这三个部分（视图、会话、Viminfo）中的最小子集，它是单个窗口相关设置的集合。如果您长时间在一个窗口上工作，并且想要保留其映射和折叠，您可以使用视图。

我们来创建一个 `foo.txt` 文件：

```
foo1
foo2
foo3
foo4
foo5
foo6
foo7
foo8
foo9
foo10
```

在这个文件中，做三次修改：

1. 在第 1 行，创建一个手动折叠 `zf4j`（折叠接下来 4 行）。
2. 更改 `number` 设置：`setlocal nonumber norelativenumber`。这会移除窗口左侧的数字指示器。
3. 创建本地映射，每当按一次 `j` 时，向下两行：`:nnoremap <buffer> j jj`。

您的文件看起来应该像：

```
+-- 5 lines: foo1 -----
foo6
foo7
foo8
foo9
foo10
```

### 配置视图属性

运行：

```
:set viewoptions?
```

默认情况下会显示（根据您的 vimrc 可能会有所不同）：

```
viewoptions=folds,cursor,curdir
```

我们来配置 `viewoptions`。要保留的三个属性分别是折叠、映射和本地设置选项。如果您的设置和我的相似，那么您已经有了 `folds` 选项。运行下列命令使视图记住 `localoptions`：

```
:set viewoptions+=localoptions
```

查阅 `:h viewoptions` 可了解 `viewoptions` 的其他可用选项。现在运行 `:set viewoptions?`，您将看到：

```
viewoptions=folds,cursor,curdir,localoptions
```

### 保存视图

在 `foo.txt` 窗口经过适当折叠并设置了 `nonumber norelativenumber` 选项后，现在我们来保存视图。运行：

```
:mkview
```

Vim 创建了一个视图文件。

### 视图文件

您可能会想“Vim 将这个视图文件保存到哪儿了呢？”，运行下列命令就可以看到答案了：

```
:set viewdir?
```

默认情况下会显示 `~/.vim/view`（根据您的操作系统，可能会有不同的路径。查阅 `:h viewdir` 获得更多信息）。如果您运行的是基于Unix的操作系统，想修改该路径，可以在您的 vimrc 中添加下列内容：

```
set viewdir=$HOME/else/where
```

### 加载视图文件

关闭并重新打开 `foo.txt`，您会看到原来的文本，没有任何改变。这是预期行为。运行下列命令可以加载视图文件：

```
:loadview
```

现在您将看到：

```
+-- 5 lines: foo1 -----
foo6
foo7
foo8
foo9
foo10
```

那些折叠、本地设置以及映射都恢复了。如果您细心还可以发现，光标位于上一次您运行 `:mkview` 时所处的行上。只要您有 `cursor` 选项，视图将记住光标位置。

### 多个视图

Vim 允许您保存 9 个编号的视图（1-9）。

假设您想用 `:9,10 fold` 来额外折叠最后两行，我们把这存为视图 1。运行：

```
:mkview 1
```

如果您又想用 `:6,7 fold` 再折叠一次，并存为不同的视图，运行：

```
:mkview 2
```

关闭并重新打开 `foo.txt` 文件，运行下列命令可以加载视图 1：

```
:loadview 1
```

要加载视图 2，运行：

```
:loadview 2
```

要加载原始视图，运行：

```
:loadview
```

### 自动创建视图

有一件可能会发生的很倒霉的事情是，您花了很长时间在一个大文件中进行折叠，一不小心关闭了窗口，接着丢失了所有折叠信息。您可以在 vimrc 中添加下列内容，使得在关闭缓冲区后 Vim 能自动创建视图，防止此类灾难发生：

```
autocmd BufWinLeave *.txt mkview
```

另外也能在打开缓冲区后自动加载视图：

```
autocmd BufWinEnter *.txt silent loadview
```

现在，当您编辑 `txt` 文件时，不用再担心创建和加载视图了。但也注意，随着时间的推移，视图文件会不断积累，记得每隔几个月清理一次。

## 会话

如果说视图保存了某个窗口的设置，那么会话则保存了所有窗口（包括布局）的信息。

### 创建新会话

假设您在 `foobarbaz` 工程中编辑着 3 个文件：

`foo.txt` 的内容：

```
foo1
foo2
foo3
foo4
foo5
foo6
foo7
foo8
foo9
foo10
```

`bar.txt` 的内容：

```
bar1
bar2
bar3
bar4
bar5
bar6
bar7
bar8
bar9
bar10
```

`baz.txt` 的内容：

```
baz1
baz2
baz3
baz4
baz5
baz6
baz7
baz8
baz9
baz10
```

假设您的窗口布局如下所示（适当地使用 `split` 和 `vsplit` 来放置）：

![Session Layout](images/session-layout.png)

要保留这个外观，您需要保存会话。运行：

```
:mksession
```

与默认存储在 `~/.vim/view` 的 `mkview` 不同，`mksession` 在当前目录存储会话文件（`Session.vim`）。如果好奇，您可以看看文件。

如果您想将会话文件另存他处，可以将参数传递给 `mksession`：

```
:mksession ~/some/where/else.vim
```

使用 `!` 来调用命令可以覆盖一个已存在的会话文件（`:mksession! ~/some/where/else.vim`）。

### 加载会话

运行下列命令可以加载会话：

```
:source Session.vim
```

现在 Vim 看起来就像您离开它时的样子！或者，您也可以从终端加载会话文件：

```
vim -S Session.vim
```

### 配置会话属性

您可以配置会话要保存的属性。若要查看当前哪些属性正被保存，请运行：

```
:set sessionoptions?
```

我的显示：

```
blank,buffers,curdir,folds,help,tabpages,winsize,terminal
```

如果在保存会话时不想存储 `terminal`，可以运行下列命令将其从会话选项中删除：

```
:set sessionoptions-=terminal
```

如果要在保存会话时存储 `options`，请运行：

```
:set sessionoptions+=options
```

下面是一些 `sessionoptions` 可以存储的属性：

- `blank` 存储空窗口
- `buffers` 存储缓冲区
- `folds` 存储折叠
- `globals` 存储全局变量（必须以大写字母开头，并且至少包含一个小写字母）
- `options` 存储选项和映射
- `resize` 存储窗口行列
- `winpos` 存储窗口位置
- `winsize` 存储窗口大小
- `tabpages` 存储选项卡
- `unix` 以 Unix 格式存储文件

查阅 `:h 'sessionoptions'` 来获取完整列表。

会话是保存项目外部属性的好工具。但是，一些内部属性不存储在会话中，如本地标记、寄存器、历史记录等。要保存它们，您需要使用 Viminfo！

## Viminfo

如果您留意，在复制一个单词进寄存器 a，再退出并重新打开 Vim 后，您仍然可以看到存储在寄存器中的文本。这就是 Viminfo 的功劳。没有它，在您关闭 Vim 后，Vim 会忘记这些寄存器。

如果您使用 Vim 8 或更高版本，Vim 会默认启用 Viminfo。因此您可能一直在使用 Viminfo，而您对它毫不知情！

您可能会问：Viminfo 存储了什么？与会话有何不同？

要使用 Viminfo，您必须启用了 `+viminfo` 特性（`:version`）。Viminfo 存储着：

- 命令行历史记录。
- 字符串搜索历史记录。
- 输入行历史记录。
- 非空寄存器的内容。
- 多个文件的标记。
- 文件标记，它指向文件中的位置。
- 上次搜索 / 替换模式（用于 “n” 和 “&”）。
- 缓冲区列表。
- 全局变量。

通常，会话存储“外部”属性，Viminfo 存储“内部”属性。

每个项目可以有一个会话文件，而 Viminfo 与会话不同，通常每台计算机只使用一个 Viminfo。Viminfo 是项目无关的。

对于 Unix，Viminfo 的默认位置是 `$HOME/.viminfo`（`~/.viminfo`）。如果您用其他操作系统，Viminfo 位置可能会有所不同。可以查阅 `:h viminfo-file-name`。每一次您做出的“内部”更改，如将文本复制进一个寄存器，Vim 都会自动更新 Viminfo 文件。

*请确保您设置了 `nocompatible` 选项（`set nocompatible`），否则您的 Viminfo 将不起作用。*

### 读写 Viminfo

尽管只使用一个 Viminfo 文件，但您还是可以创建多个 Viminfo 文件。使用 `:wviminfo` 命令（缩写为 `:wv`）来创建多个 Viminfo 文件。

```
:wv ~/.viminfo_extra
```

要覆盖现有的 Viminfo 文件，向 `wv` 命令多添加一个叹号：

```
:wv! ~/.viminfo_extra
```

Vim 默认情况下会读取 `~/.viminfo` 文件。运行 `:rviminfo`（缩写为 `:rv`）可以读取不同的 Vimfile 文件：

```
:rv ~/.viminfo_extra
```

要在终端使用不同的 Viminfo 文件来启动 Vim，请使用 “i” 标志：

```
vim -i viminfo_extra
```

如果您要将 Vim 用于不同的任务，比如写代码和写作，您可以创建两个 Viminfo，一个针对写作优化，另一个为写代码优化。

```
vim -i viminfo_writing

vim -i viminfo_coding
```

### 不使用 Viminfo 启动 Vim

要不使用 Viminfo 启动 Vim，可以在终端运行：

```
vim -i NONE
```

要永不使用 Viminfo，可以在您的 vimrc 文件添加：

```
set viminfo="NONE"
```

### 配置 Viminfo 属性

和 `viewoptions` 以及 `sessionoptions` 类似，您可以用 `viminfo` 选项指定要存储的属性。请运行：

```
:set viminfo?
```

您会得到：

```
!,'100,<50,s10,h
```

看起来有点晦涩难懂。命令分解如下：
- `!` 保存以大写字母开头、却不包含小写字母的全局变量。回想一下 `g:` 代表了一个全局变量。例如，假设您写了赋值语句 `let g:FOO = "foo"`，Viminfo 将存储全局变量 `FOO`。然而如果您写了 `let g:Foo = "foo"`，Viminfo 将不存储它，因为它包含了小写字母。没有 `!`，Vim 不会存储这些全局变量。
- `'100` 代表标记。在这个例子中，Viminfo 将保存最近 100 个文件的本地标记（a-z）。注意，如果存储的文件过多，Vim 会变得很慢，1000 左右就可以了。
- `<50` 告诉 Viminfo 每个寄存器最多保存多少行（这个例子中是 50 行）。如果我复制 100 行文本进寄存器 a（`"ay99j`）后关闭 Vim，下次打开 Vim 并从寄存器 a（`"ap`）粘贴时，Vim 最多只粘贴 50 行；如果不指定最大行号， *所有* 行都将被保存；如果指定 0，什么都不保存了。
- `s10` 为寄存器设置大小限制（kb）。在这个例子中，任何大于 10kb 的寄存器都会被排除。
- `h` 禁用高亮显示（`hlsearch` 时）。

可以查阅 `:h 'viminfo'` 来了解其他更多选项。

## 聪明地使用视图、会话和 Viminfo

Vim 能使用视图、会话和 Viminfo 来保存不同级别的 Vim 环境快照。对于微型项目，可以使用视图；对于大型项目，可以使用会话。您应该花些时间来查阅视图、会话和 Viminfo 提供的所有选项。

为您的编辑风格创建属于您自己的视图、会话和 Viminfo。如果您要换台计算机使用 Vim，只需加载您的设置，立刻就会感到就像在家里的工作环境一样！

## 链接
- [目录](./directory.md)
- 上一部分 [Ch 19 - 编译](./ch19_compile.md)
- 下一部分 [Ch 21 - 多文件操作](./ch21_multiple_file_operations.md)