# 第16章 标签

快速转到任意定义处，是文本编辑中一个非常有用的特性。在本章中，您将学习如何使用 Vim 标签来做到这一点。

## 标签概述

假设有人给了您一个新的代码库：

```
one = One.new
one.donut
```

`One`？`donut`？呃，对于当时编写代码的开发者而言，这些代码的含义可能显而易见。问题是当时的开发者已经不在了，现在要由您来理解这些费解的代码。而跟随有`One` 和 `donut`定义的源代码，是帮助您理解的一个有效方法。

您可以使用`fzf` 或 `grep`来搜索它们，但这种情况下，但使用标签将更快。

把标签想象成地址簿：

```
Name    Address
Iggy1   1234 Cool St, 11111
Iggy2   9876 Awesome Ave, 2222
```

当然，标签可不是存储着“姓名-地址”对，而是“定义-地址”对。

假设您在一个目录中有两个 Ruby 文件：

```
## one.rb
class One
  def initialize
    puts "Initialized"
  end

  def donut
    puts "Bar"
  end
end
```

以及

```
## two.rb
require './one'

one = One.new
one.donut
```

在普通模式下，您可以使用`Ctrl-]`跳转到定义。在`two.rb`中，转到`one.donut`所在行，将光标移到`donut`处，按下`Ctrl-]`。

哦豁，Vim 找不到标签文件，您需要先生成它。

## 标签生成器

现代 Vim 不自带标签生成器，您需要额外下载它。有几个选项可供选择：

- ctags = 仅用于 C，基本随处可见。
- exuberant ctags = 最流行的标签生成器之一，支持许多语言。
- universal ctags = 和 exuberant ctags 类似，但比它更新。
- etags = 用于 Emacs，嗯……
- JTags = Java
- ptags.py = Python
- ptags = Perl
- gnatxref = Ada

如果您查看 Vim 在线教程，您会发现许多都会推荐 [exuberant ctags](http://ctags.sourceforge.net/)，它支持 [41 种编程语言](http://ctags.sourceforge.net/languages.html)，我用过它，挺不错的。但自2009年以来一直没有维护，因此 Universal ctags 更好些，它和 exuberant ctags 相似，并仍在维护。

我不打算详细介绍如何安装 Universal ctags，您可以在 [universal ctags](https://github.com/universal-ctags/ctags) 仓库了解更多说明。

假设您已经安装好了ctags，接下来，生成一个基本的标签文件。运行：

```
ctags -R .
```

 `R` 选项告诉 `ctags` 从当前位置 (`.`) 递归扫描文件。稍后，您应该在当前文件夹看到一个`tags` 文件，里面您将看到类似这样的内容：

```
!_TAG_FILE_FORMAT	2	/extended format; --format=1 will not append ;" to lines/
!_TAG_FILE_SORTED	1	/0=unsorted, 1=sorted, 2=foldcase/
!_TAG_OUTPUT_FILESEP	slash	/slash or backslash/
!_TAG_OUTPUT_MODE	u-ctags	/u-ctags or e-ctags/
!_TAG_PATTERN_LENGTH_LIMIT	96	/0 for no limit/
!_TAG_PROGRAM_AUTHOR	Universal Ctags Team	//
!_TAG_PROGRAM_NAME	Universal Ctags	/Derived from Exuberant Ctags/
!_TAG_PROGRAM_URL	<https://ctags.io/>	/official site/
!_TAG_PROGRAM_VERSION	0.0.0	/b43eb39/
One	one.rb	/^class One$/;"	c
donut	one.rb	/^  def donut$/;"	f	class:One
initialize	one.rb	/^  def initialize$/;"	f	class:One
```

根据 Vim 设置和 ctag 生成器的不同，您的`tags` 文件可能会有些不同。一个标签文件由两部分组成：标签元数据和标签列表。那些标签元数据 (`!TAG_FILE...`) 通常由 ctags 生成器控制。这里我不打算介绍它们，您可以随意查阅文档。标签列表是一个由所有定义组成的列表，由ctags建立索引。

现在回到 `two.rb`，将光标移至 `donut`，再输入`Ctrl-]`，Vim 将带您转到 `one.rb` 文件里`def donut` 所在的行上。成功啦！但 Vim 怎么做到的呢？

## 解剖标签文件

来看看`donut` 标签项：

```
donut	one.rb	/^  def donut$/;"	f	class:One
```

上面的标签项由四个部分组成：一个`tagname`、一个`tagfile`、一个`tagaddress`，以及标签选项。

- `donut` 是 `tagname`。当光标在 "donut" 时，Vim 搜索标签文件里含有 "donut" 字符串的一行。
- `one.rb` 是 `tagfile`。Vim 会搜寻 `one.rb` 文件。
- `/^ def donut$/` 是 `tagaddress`。`/.../` 是模式指示器。`^` 代表一行中第一个元素，后面跟着两个空格，然后是`def donut`字符串，最后 `$` 代表一行中最后一个元素。
- `f class:One` 是标签选项，它告诉 Vim，`donut` 是一种函数 (`f`)，并且是 `One` 类的一部分。

再看看另一个标签项：

```
One	one.rb	/^class One$/;"	c
```

这一行和 `donut`也是一样的：

- `One` 是 `tagname`。注意，对于标签，第一次扫描区分大小写。如果列表中有 `One` 和 `one`， Vim 会优先考虑 `One` 而不是 `one`。
- `one.rb` 是 `tagfile`。Vim 会搜寻 `one.rb` 文件。
- `/^class One$/` 是 `tagaddress` 。Vim 会查找以 `class` 开头 (`^`) 、以 `One` 结尾 (`$`) 的行。
- `c` 是可用标签选项之一。由于 `One` 是一个 ruby 类而不是过程，因此被标签为 `c`。

标签文件的内容可能不尽相同，根据您使用的标签生成器而定。但至少，标签文件必须具有以下格式之一：

```
1.  {tagname} {TAB} {tagfile} {TAB} {tagaddress}
2.  {tagname} {TAB} {tagfile} {TAB} {tagaddress} {term} {field} ..
```

## 标签文件

您知道，在运行 `ctags -R .` 后，一个新 `tags` 文件会被创建。但是，Vim 是如何知道在哪儿查找标签文件的呢？

如果运行 `:set tags?`，您可能会看见 `tags=./tags,tags`（根据您的 Vim 设置，内容可能有所不同）。对于 `./tags`，Vim 会在当前文件所在路径查找所有标签；对于 `tags`，Vim 会在当前目录（您的项目根路径）中查找。

此外，对于 `./tags`，Vim 会在当前文件所在路径内查找一个标签文件，无论它被嵌套得有多深。接下来，Vim 会在当前目录（项目根路径）查找。Vim 在找到第一个匹配项后会停止搜索。

如果您的 `'tags'` 文件是 `tags=./tags,tags,/user/iggy/mytags/tags`，那么 Vim 在搜索完 `./tags` 和 `tags` 目录后，还会在 `/user/iggy/mytags` 目录内查找。所以您可以分开存放标签文件，不必将它们置于项目文件夹中。

要添加标签文件位置，只需要运行：

```
:set tags+=path/to/my/tags/file
```

## 为大型项目生成标签：

如果您尝试在大型项目中运行 ctag，则可能需要很长时间，因为 Vim 也会查看每个嵌套目录。如果您是 Javascript 开发者，您会知道 `node_modules` 非常大。假设您有五个子项目，每个都包含自己的 `node_modules` 目录。一旦运行 `ctags -R .`，ctags 将尝试扫描这5个 `node_modules`。但您可能不需要为 `node_modules` 运行 ctag。

如果要排除 `node_modules` 后执行 ctags，可以运行：

```
 ctags -R --exclude=node_modules .
```

这次应该只需要不到一秒钟的时间。另外，您还可以多次使用 `exclude` 选项：

```
ctags -R --exclude=.git --exclude=vendor --exclude=node_modules --exclude=db --exclude=log .
```

## 标签导航

仅使用 `Ctrl-]` 也挺好，但我们还可以多学几个技巧。其实，标签跳转键 `Ctrl-]` 还有命令行模式：`:tag my-tag`。如果您运行：

```
:tag donut
```

Vim 就会跳转至 `donut` 方法，就像在 "donut" 字符串上按 `Ctrl-]` 一样。您还可以使用 `<Tab>` 来自动补全参数：

```
:tag d<Tab>
```

Vim 会列出所有以 "d" 开头的标签。对于上面的命令，结果则是 "donut"。

在实际项目中，您可能会遇到多个同名的方法。我们来更新下这两个文件。先是 `one.rb`：

```
## one.rb
class One
  def initialize
    puts "Initialized"
  end

  def donut
    puts "one donut"
  end

  def pancake
    puts "one pancake"
  end
end
```

然后 `two.rb`：

```
## two.rb
require './one.rb'

def pancake
  "Two pancakes"
end

one = One.new
one.donut
puts pancake
```

由于新添加了一些过程，因此编写完代码后，不要忘记运行 `ctags -R .`。现在，您有了两个 `pancake` 过程。如果您在 `two.rb` 内按下 `Ctrl-]`，会发生什么呢？

Vim 会跳转到 `two.rb` 内的 `def pancake`，而不是 `one.rb` 的 `def pancake`。这是因为 Vim 认为 `two.rb` 内部的 `pancake` 过程比其他的`pancake` 过程具有更高优先级。

## 标签优先级

并非所有的标签都有着相同的地位。一些标签有着更高的优先级。如果有重复的标签项，Vim 会检查关键词的优先级。顺序是：

1. 当前文件中完全匹配的静态标签。
2. 当前文件中完全匹配的全局标签。
3. 其他文件中完全匹配的全局标签。
4. 其他文件中完全匹配的静态标签。
5. 当前文件中不区分大小写匹配的静态标签。
6. 当前文件中不区分大小写匹配的全局标签。
7. 其他文件中区分大小写匹配的全局标签。
8. 当前文件中不区分大小写匹配的静态标签。

根据优先级列表，Vim 会对在同一个文件上找到的精确匹配项进行优先级排序。这就是为什么 Vim 会选择 `two.rb` 里的 `pancake` 过程而不是 `one.rb` 里的。但是，上述优先级列表有些例外，取决于您的`'tagcase'`、`'ignorecase'`、`'smartcase'` 设置。我不打算介绍它们，您可以自行查阅 `:h tag-priority`。

## 选择性跳转标签

如果可以选择要跳转到哪个标签，而不是始终转到优先级最高的，那就太好了。因为您可能想跳转到 `one.rb` 里的 `pancake` 方法，而不是 `two.rb` 里的。现在您可以使用 `:tselect` 做到它！运行：

```
:tselect pancake
```

您可以在屏幕底部看到：

```
## pri kind tag               file
1 F C f    pancake           two.rb
             def pancake
2 F   f    pancake           one.rb
             class:One
             def pancake
```

如果输入`2` 后再 `<Return>`，Vim 将跳转到 `one.rb` 里的`pancake` 过程。如果输入`1` 后再 `<Return>`，Vim 将跳转到 `two.rb` 里的。

注意`pri` 列，第一个匹配中该列是`F C`，第二个匹配中则是`F`。这就是 Vim 用来确定标签优先级的凭据。`F C`表示在当前 (`C`) 文件中完全匹配 (`F`) 的全局标签。`F` 表示仅完全匹配 (`F`) 的全局标签。`F C` 的优先级永远比 `F` 高。*（译注：`F`是`Fully-matched`，`C`是`Current file`）*

如果运行`:tselect donut`，即使只有一个标签可选，Vim 也会提示您选择跳转到哪一个。有没有什么方法可以让 Vim 仅在有多个匹配项时才提示标签列表，而只找到一个标签时就立即跳转呢？

当然！Vim 有一个 `:tjump` 方法。运行：

```
:tjump donut
```

Vim 将立即跳转到 `one.rb` 里的`donut` 过程，就像在运行 `:tag donut` 一样。现在试试：

```
:tjump pancake
```

Vim 将提示您从标签选项中选择一个，就像在运行`:tselect pancake`。`tjump` 能两全其美。

`tjump` 在普通模式下有一个快捷键：`g Ctrl-]`。我个人喜欢`g Ctrl-]`胜过 `Ctrl-]`。

## 标签的自动补全

标签能有助于自动补全。回想下第6章“插入模式”，您可以使用 `Ctrl-x` 子模式来进行各式自动补全。其中有一个我没有提到过的自动补全子模式便是 `Ctrl-]`。如果您在插入模式中输入`Ctrl-x Ctrl-]`，Vim 将使用标签文件来自动补全。

在插入模式下输入`Ctrl-x Ctrl-]`，您会看到：

```
One
donut
initialize
pancake
```

## 标签堆栈

Vim 维持着一个标签堆栈，上面记录着所有您从哪儿来、跳哪儿去的标签列表。使用 `:tags` 可以看到这个堆栈。如果您首先跳转到`pancake`，紧接着是`donut`，此时运行`:tags`，您将看到：

```
  # TO tag         FROM line  in file/text
  1  1 pancake            10  ch16_tags/two.rb
  2  1 donut               9  ch16_tags/two.rb
>
```

注意上面的 `>` 符号，它代表着您当前在堆栈中的位置。要“弹出”堆栈，从而回到上一次的状态，您可以运行`:pop`。试试它，再运行`:tags`看看：

```
  # TO tag         FROM line  in file/text
  1  1 pancake            10  puts pancake
> 2  1 donut               9  one.donut

```

注意现在 `>` 符号位于 `donut` 所在的第二行了。再 `pop` 一次，然后运行`:tags`：

```
  # TO tag         FROM line  in file/text
> 1  1 pancake            10  puts pancake
  2  1 donut               9  one.donut
```

在普通模式下，您可以按下 `Ctrl-t` 来达到和 `:pop` 一样的效果。

## 自动生成标签

Vim 标签最大的缺点之一是，每当进行重大改变时，您需要重新生成标签文件。如果您将`pancake` 过程重命名为 `waffle`，标签文件不知道 `pancake` 被重命名了，标签列表仍旧存储着 `pancake` 过程。运行`ctags -R .` 可以创建更新的标签文件，但这可能会很缓慢。

幸运的是，有几种可以自动生成标签的方法。这一小节不打算介绍一个简单明了的过程，而是提出一些想法，以便您可以扩展它们。

## 在保存时生成标签

Vim 有一个自动命令 (`autocmd`) 方法，可以在触发事件时执行任意命令。您可以使用这个方法，以便在每次保存时生成标签。运行：

```
:autocmd BufWritePost *.rb silent !ctags -R .
```

上面命令的分解如下：

- `autocmd` 是 Vim 的自动命令方法，它接受一个事件名称、文件和一个命令。
- `BufWritePost` 是保存缓冲区时的一个事件。每次保存文件时将触发一次 `BufWritePost` 事件。
- `.rb` 是 ruby (`rb`) 文件的一种文件模式。
- `silent` 是您传递的命令的一部分。如果不输入它，每次触发自动命令时，Vim 都会提示  `press ENTER or type command to continue`。
- `!ctags -R .` 是要执行的命令。回想一下，`!cmd` 从 Vim 内部执行终端命令。

现在，每次您保存一个 ruby 文件时，Vim 都会运行`ctags -R .`。

## 使用插件

有几种插件可以自动生成 ctags：

- [vim-gutentags](https://github.com/ludovicchabant/vim-gutentags)
- [vim-tags](https://github.com/szw/vim-tags)
- [vim-easytags](https://github.com/xolox/vim-easytags)
- [vim-autotag](https://github.com/craigemery/vim-autotag)

我使用 vim-gutentags。它的使用方法很简单，而且装上就可以直接使用。

## Ctags 以及 Git 钩子

Tim Pope 是一个写了很多非常棒的 Vim 插件的作者，他写了一篇博客，建议使用 git 钩子。[可以看一看](https://tbaggery.com/2011/08/08/effortless-ctags-with-git.html)。

## 聪明地学习标签

只要配置得当，标签是非常有用的。假设在一个新的代码库中，您想要搞清楚 `functionFood` 干了什么，您可以通过跳转到它的定义来搞懂它们。在那儿可以看到，它又调用了 `functionBreakfast`。继续跟踪，发现还调用了 `functionPancake`。现在您明白了，函数调用路径图长这样：

```
functionFood -> functionBreakfast -> functionPancake
```

进一步可以知道，这段代码和早餐吃煎饼有关。

现在您已经知道如何使用标签，通过 `:h tags` 可以学习更多有关标签的知识。接下来让我们一起来探索另一个功能：折叠。

## 链接
- [目录](./directory.md)
- 上一部分 [Ch 15 - 命令行模式](./ch15_command-line_mode.md)
- 下一部分 [Ch 17 - 折叠](./ch17_fold.md)