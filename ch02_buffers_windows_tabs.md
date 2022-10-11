# 第2章 缓冲区(Buffers)，窗口(Windows)和选项卡(Tabs)

(译者注：在Vim中，Buffers缓冲区，Windows窗口，Tabs选项卡是专有名词。为适应不同读者的翻译习惯，确保没有歧义，本文将不对Buffers、Windows、Tabs这三个词进行翻译)。  

如果您使用过现代文本编辑器，您很可能对Windows和tabs这两个概念是非常熟悉的。但Vim使用了三个关于显示方面的抽象概念：buffers, windows, 还有tabs。在本章，我将向您解释什么是buffers, windows和tabs，以及它们在Vim中如何工作。

在开始之前，确保您的vimrc文件中开启了`set hidden`选项。若没有配置该选项，当您想切换buffer且当前buffer没有保存时，Vim将提示您保存文件（如果您想快速切换，您不会想要这个提示）。我目前还没有讲vimrc，如果您没有vimrc配置文件，那就创建一个。它通常位于根目录下，名字叫`.vimrc`。我的vimrc位于`~/.vimrc`。要查看您自己的vimrc文件应该放置在哪，可以在Vim命令模式中输入`:h vimrc`。在vimrc文件中，添加：

```
set hidden
```

保存好vimrc文件，然后激活它(在vimrc文件中运行`:source %`)。

## Buffers

*buffer*到底是什么？

buffer就是内存中的一块空间，您可以在这里写入或编辑文本。当您在Vim中打开一个文件时，文件的数据就与一个buffer绑定。当您在Vim中打开3个文件，您就有3个buffers。

创建两个可使用的空文件，分别名为`file1.js`和`file2.js`（如果可能，尽量使用Vim来创建）。在终端中运行下面的命令：

```bash
vim file1.js
```

这时您看到的是`file1.js`的 *buffer* 。每当您打开一个新文件，Vim总是会创建一个新的buffer。

退出Vim。这一次，打开两个新文件：

```bash
vim file1.js file2.js
```

Vim当前显示的是`file1.js`的buffer，但它实际上创建了两个buffers：`file1.js`buffer和`file2.js`buffer。运行`:buffers`命令可以查看所有的buffers（另外，您也可以使用`:ls`和`:files`命令）。您应该会 *同时* 看到列出来的`file1.js`和`file2.js`。运行`vim file1 file2 file3 ... filen`创建n个buffers。每一次您打开一个新文件，Vim就为这个文件创建一个新的buffer。

要遍历所有buffers，有以下几种方法：
- `:bnext` 切换至下一个buffer（`:bprevious`切换至前一个buffer）。
- `:buffer` + 文件名。（按下`<Tab>`键Vim会自动补全文件名）。 
- `:buffer` + `n`, n是buffer的编号。比如，输入`:buffer 2`将使您切换到buffer #2。
- 按下`Ctrl-O`将跳转至跳转列表中旧的位置，对应的，按下`Ctrl-I`将跳转至跳转列表中新的位置。这并不是属于buffer的特有方法，但它可以用来在不同的buffers中跳转。我将在第5章详细讲述关于跳转的知识。
- 按下`Ctrl-^`跳转至先前编辑过的buffer。

一旦Vim创建了一个buffer，它将保留在您的buffers列表中。若想删除它，您可以输入`:bdelete`。这条命令也可以接收一个buffer编号（`:bdelete 3`将删除buffer #3）或一个文件名（`:bdelete`然后按`<Tab>`自动补全文件名）。

我学习buffer时最困难的事情就是理解buffer如何工作，因为我当时的思维已经习惯了使用主流文本编辑器时关于窗口的概念。要理解buffer，可以打个很好的比方，就是打牌的桌面。如果您有2个buffers，就像您有一叠牌（2张）。您只能看见顶部的牌，虽然您知道在它下面还有其他的牌。如果您看见`file1.js`buffer，那么`file1.js`就是顶部的牌。虽然您看不到其他的牌`file2.js`，但它实际上就在那。如果您切换buffers到`file2.js`，那么`file2.js`这张牌就换到了顶部，而`file1.js`就换到了底部。

如果您以前没有用过Vim，这是一个新的概念。花上几分钟理解一下。

## 退出Vim

顺带说一句，如果您已经打开了多个buffers，您可以使用quit -all来关闭所有的buffers：

```
:qall
```

如果您想关闭所有buffers但不保存，仅需要在后面加`!`（叹号）就行了：

```
:qall!
```

若要保存所有buffers然后退出，请运行：

```
:wqall
```

## Windows

一个window就是在buffer上的一个视口。如果您使用过主流的编辑器，Windows这个概念应该很熟悉。大部分文本编辑器具有显示多个窗口的能力。在Vim中，您同样可以拥有多个窗口。

让我们从终端再次打开`file1.js`：

```bash
vim file1.js
```

先前我说过，您看到的是`file1.js`的buffer。但这个说法并不完整，现在这句话得更正一下，您看到的是`file1.js `的buffer通过 **一个窗口** 显示出来。窗口就是您查看的buffer所使用的视口。

先不忙急着退出Vim，在Vim中运行：

```
:split file2.js
```

现在您看到的是两个buffers通过 **两个窗口** 显示出来。上面的窗口显示的是`file2.js`的buffer。而下面的窗口显示的是`file1.js`的buffer。

如果您想在窗口之间导航，使用这些快捷键：

```
Ctrl-W H    移动光标到左边的窗口
Ctrl-W J    移动光标到下面的窗口
Ctrl-W K    移动光标到上面的窗口
Ctrl-W L    移动光标到右边的窗口
```

现在，在Vim中运行：

```
:vsplit file3.js
```

您现在看到的是三个窗口显示三个buffers。一个窗口显示`file3.js`的buffer，一个窗口显示`file2.js`的buffer，还有一个窗口显示`file1.js`的buffer。

您可以使多个窗口显示同一个buffer。当光标位于左上方窗口时，输入：

```
:buffer file2.js
```

现在两个窗口显示的都是`file2.js`的buffer。如果您现在在这两个窗口中的某一个输入内容，您会看到所有显示`file2.js`buffer的窗口都在实时更新。

要关闭当前的窗口，您可以按`Ctrl-W C`或输入`:quit`。当您关闭一个窗口后，buffers仍然会在列表中。（可以运行`:buffers`来确认这一点）。

这里有一些普通模式下关于窗口的命令：

```
Ctrl-W V    打开一个新的垂直分割的窗口
Ctrl-W S    打开一个新的水平分割的窗口
Ctrl-W C    关闭一个窗口
Ctrl-W O    除了当前窗口，关闭所有其他的窗口
```

另外，下面的列表列出了一些有用的关于windows的命令行命令

```
:vsplit filename    垂直分割当前窗口，并在新窗口中打开名为filename的文件。
:split filename     水平分割当前窗口，并在新窗口中打开名为filename的文件。
:new filename       创建一个新窗口并打开名为filename的文件。
```

花一点时间理解上面的知识。要了解更多信息，可以查看帮助`:h window`。

## Tabs

Tabs就是windows的集合。它就像窗口的布局。在大部分的现代文本编辑器（还有现代互联网浏览器）中，一个tab意味着打开一个文件/页面，当您关闭标签，相应的文件/页面就消失了。但在Vim中，tab并不表示打开了一个文件。当您在Vim中关闭一个tab，您并不是关闭一个文件。您仅仅关闭了窗口布局。文件的数据依然存储在内存中的buffers中。

让我们运行几个命令看看Vim中tabs的功能。打开`file1.js`：

```bash
vim file1.js
```

若要在新tab中打开`file2.js`：

```
:tabnew file2.js
```

当然您可以按`<Tab>`让Vim自动补全 *新tab* 中将要打开的文件名（啰嗦几句，请理解作者的幽默 ）。

下面的列表列出了一些有用的关于tab导航的命令：

```
:tabnew file.txt    在tab中打开一个文件
:tabclose           关闭当前tab
:tabnext            切换至下一个tab
:tabprevious        切换至前一个tab
:tablast            切换至最后一个tab
:tabfirst           切换至第一个tab
```

您可以输入`gt`切换到下一个标签页（对应的，可以用`gT`切换到前一个标签页）。您也可以传递一个数字作为参数给`gt`，这个数字是tab的编号。若想切换到第3个tab，输入`3gt`。

拥有多个tabs的好处是，您可以在不同的tab中使用不同的窗口布局。也许，您想让您的第1个tab包含3个垂直分割的窗口，然后让第2个tab为水平分割和垂直分割混合的窗口布局。tab是完成这件工作的完美工具!

若想让Vim启动时就包含多个tabs，您可以在终端中运行如下命令：

```bash
vim -p file1.js file2.js file3.js
```

## 三维移动

在windows之间移动就像在笛卡尔坐标系的二维平面上沿着X-Y轴移动。您可以使用`Ctrl-W H/J/K/L`移动到上面、右侧、下面、以及左侧的窗口。

在buffer之间移动就像在笛卡尔坐标系的Z轴上穿梭。想象您的buffer文件在Z轴上呈线性排列，您可以使用`:bnext`和`bprevious`在Z轴上一次一个buffer地遍历。您也可以使用`:buffer 文件名/buffer编号`在Z轴上跳转到任意坐标。

结合window和buffer的移动，您可以在 *三维空间* 中移动。您可以使用window导航命令移动到上面、右侧、下面、或左侧的窗口（X-Y平面导航）。因为每个window都可能包含了多个buffers，您可以使用buffer移动命令向前、向后移动（Z轴导航）。

## 用聪明的方法使用Buffers、Windows、以及Tabs

您已经学习了什么是buffers、windows、以及tabs，也学习了它们如何在Vim中工作。现在您对它们有了更好地理解，您可以把它们用在您自己的工作流程中。

每个人都有不同的工作流程，以下示例是我的工作流程：
- 首先，对于某个特定任务，我先使用buffers存储所有需要的文件。Vim就算打开很多buffer，速度一般也不会减慢。另外打开多个buffers并不会使我的屏幕变得拥挤。我始终只会看到1个buffer（假设我只有1个window），这可以让我注意力集中在1个屏幕上。当我需要使用其他文件时，可以快速切换至对应文件的buffer。
- 当比对文件、读文档、或追踪代码流时，我使用多窗口来一次查看多个buffers。我尽量保持屏幕上的窗口数不超过3个，因为超过3个屏幕将变得拥挤（我使用的是小型笔记本）。当相应工作完成后，我就关掉多余的窗口。窗口越少可以使注意力更集中。
- 我使用[tmux](https://github.com/tmux/tmux/wiki)windows来代替tabs。通常一次使用多个tmux窗口。比如，一个tmux窗口用来写客户端代码，一个用来写后台代码。 

由于编辑风格不同，我的工作流程可能和您的工作流程不同，这没关系。您可以在实践中去探索适合您自己工作流程的编码风格。

## 链接
- [目录](./directory.md)
- 上一部分 [Ch 1 - 起步](./ch01_starting_vim.md)
- 下一部分 [Ch 3 - 打开和搜索文件](./ch03_searching_files.md)