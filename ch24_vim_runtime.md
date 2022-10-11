# 第24章 Vim Rumtime

在前面的章节中，我提到Vim会自动查找一些特殊的路径，比如在`~/.vim/` 中的 `pack/`(第23章) `compiler/`（第19章）。这些都是Vim runtime路径的例子。

除了上面提到的两个，Vim还有更多runtime路径。在本章，您将学习关于Vim runtime路径的高层次概述。本章的目标是向您展示它们什么时候被调用。知道这些知识能够帮您更进一步理解和定制Vim。

## Runtime路径

在一台Unix机器中，其中一个vim runtime路径就是 `$HOME/.vim/` （如果您用的是其他操作系统，比如Windows，您的路径可能有所不同）。要查看不同的操作系统有什么样的runtime路径，查阅 `:h runtimepath`。在本章，我将使用 `~/.vim/` 作为默认的runtime路径。

## Plugin脚本

Vim有一个runtime路径 plugin，每次Vim启动时都会执行这个路径中的所有脚本。不要把这个名字 "plugin" 和Vim的外部插件（external plugins，比如NERDTree, fzf.vim, 等)搞混了。

进入 `~/.vim/` 目录，然后创建 `plugin/` 子目录。 创建两个文件： `donut.vim` 和 `chocolate.vim`。

在 `~/.vim/plugin/donut.vim`里面:

```
echo "donut!"
```

在 `~/.vim/plugin/chocolate.vim`里面:

```
echo "chocolate!"
```

现在关闭Vim。下次您启动Vim，您将会看到 `"donut!"` 和 `:chocolate!` 的显示。此 plugin runtime路径可以用来执行一些初始化脚本。

## 文件类型检测

在开始之前，为保证检测能正常运行，确保在您的vimrc中至少包含了下列的行：

```
filetype plugin indent on
```

查阅 `:h filetype-overview` 了解更多信息。本质上，这条代码开启Vim的文件类型检测。

当您打开一个新的文件，Vim通常知道这个文件是什么类型。如果您有一个文件 `hello.rb`，运行 `:set filetype?` 会返回正确的相应 `filetype=ruby`。

Vim知道如何检测 "常见" 的文件类型（Ruby, Python, Javascript, 等）。但如果是一个自定义文件会怎样呢？您需要告诉Vim去检测它，并给它指派一个正确的文件类型。

有两种检测方法：使用文件名和使用文件内容

### 文件名检测

文件名检测使用一个文件的文件名来检测文件类型。当您打开 `hello.rb`文件时，Vim依靠扩展名 `.rb` 知道它是一个Ruby文件。

有两种方法实现文件名检测：一是使用 `ftdetect` runtime目录，二是使用 `filetype.vim` runtime文件。我们两个都看一看。

#### `ftdetect/`

让我们创建一个古怪（但优雅）的名字，`hello.chocodonut`。当您打开它后运行 `:set filetype?` ，因为它的后缀名不是常见的文件名，Vim不知道它是什么类型，会返回 `filetype=`。

您需要指示Vim将所有以 `.chocodonut`结尾的文件设置为 "chocodonut"类型的文件。在runtime路径根目录(`~/.vim/`)创建一个子目录，名为 `ftdetect/` 。在子目录里面，再创建一个名叫 `chocodonut.vim` 的文件（`~/.vim/ftdetect/chocodonut.vim`），在文件里面，添加：

```
autocmd BufNewFile,BufRead *.chocodonut set filetype=chocodonut
```

当您创建新buffer或打开新buffer时，事件`BufNewFile` 和 `BufRead` 就会被触发。 `*.chocodonut` 意思是只有当新打开的buffer文件名后缀是 `.chocodonut` 时事件才会被触发。最后，`set filetype=chocodonut` 命令将文件类型设置为chocodonut类型。

重启Vim。新建一个 `hello.chocodonut` 文件然后运行 `:set filetype?`。它将返回 `filetype=chocodonut`.

好极了！只要您想，您可以将任意多的文件放置在 `ftdetect/` 中。以后，如果您想扩展您的 donut 文件类型，你可以添加 `ftdetect/strawberrydonut.vim`, `ftdetect/plaindonut.vim` 等等。

在Vim中，实际上有两种方法设置文件类型。其中给一个是您刚刚使用的 `set filetype=chocodonut`。另一种方法是运行 `setfiletype chocodonut`。前一个命令 `set filetype=chocodonut` 将 *总是* 设置文件类型为chocodonut。 而后者`setfiletype chocodonut`只有当文件类型尚未设置时，才会将文件类型设置为chocodonut。

#### 文件类型文件

第二种文件类型检测需要你创建一个名为 `filetype.vim`的文件，并将它放置在根目录(`~/.vim/filetype.vim`)。在文件内添加一下内容：

```
autocmd BufNewFile,BufRead *.plaindonut set filetype=plaindonut
```

创建一个名为 `hello.plaindonut` 的文件。当你打开它后运行 `:set filetype?` Vim会显示正确的自定义文件类型 `filetype=plaindonut`。

太好了，修改生效了。另外，如果您仔细看看 `filetype.vim` ，您会发现当您打开`hello.plaindonut`时，这个文件文件运行了多次。为防止这一点，您可以添加一个守卫，让主脚本只运行一次。更新 `filetype.vim`:

```
if exists("did_load_filetypes")
  finish
endif

augroup donutfiletypedetection
  autocmd! BufRead,BufNewFile *.plaindonut setfiletype plaindonut
augroup END
```

`finish` 是一个Vim命令，用来停止执行剩余的脚本。表达式`"did_load_filetypes"` 并 *不是* 一个Vim内置函数。它实际上是`$VIMRUNTIME/filetype.vim` 中的一个全局变量。如果您好奇，运行`:e $VIMRUNTIME/filetype.vim`。您将会发现以下内容：

```
if exists("did_load_filetypes")
  finish
endif

let did_load_filetypes = 1
```

当Vim调用这个文件时，它会定义 `did_load_filetypes` 变量，并将它设置为 1 。在Vim中，1 表示真。你可以试着读完 `filetype.vim` 剩余的内容，看看您是否能够理解当Vim调用它时干了什么。

### 文件类型脚本

让我们学习如何基于文件内容检测文件类型。

假设您有一个无扩展名的文件的集合。这些文件唯一相同的地方是，第一行都是以 "donutify" 开头。您现在想给这些文件指派一个 `donut` 的文件类型。创建新文件，起名为 `sugardonut`, `glazeddonut`, 还有 `frieddonut`（没有扩展名）。在每个文件中，添加下列内容：

```
donutify
```

当您在`sugardonut`中运行 `:set filetype?`，Vim无法知道应该给这个文件指派什么文件类型，会返回 `filetype=`。

在runtime根目录，添加一个 `scripts.vim` 文件(`~/.vim/scripts.vim`)，在文件中，添加一下内容：

```
if did_filetype()
  finish
endif

if getline(1) =~ '^\\<donutify\\>'
  setfiletype donut
endif
```

函数 `getline(1)` 返回文件第一行的内容。它检查第一行是否以 "donutify" 开头。函数 `did_filetype()` 是Vim的内置函数，当一个与文件类型相关的事件发生至少一次时，它返回真。它用来做守卫，防止文件类型事件反复运行。

打开文件 `sugardonut` 然后运行 `:set filetype?`，Vim现在返回 `filetype=donut`。如果您打开另外一个donut文件 (`glazeddonut` 和 `frieddonut`)，Vim同样会将它们的文件类型定义为 `donut` 类型。

注意，`scripts.vim` 仅当Vim打开一个未知文件类型的文件时才会运行。如果Vim打开一个已知文件类型的文件，`scripts.vim` 将不会运行。

## 文件类型插件

如果您想让Vim仅当您打开一个 chocodonut 文件时才运行 chocodonut 相关的特殊脚本，而当您打开的是 plaindonut 文件时，Vim就不运行这些脚本。能否做到呢？

您可以使用文件类型插件runtime路径(`~/.vim/ftplugin/`)来完成这个功能。Vim会在这个目录中查找一个文件，这个文件的文件名与您打开的文件类型一样。创建一个文件，起名为`chocodonut.vim` (`~/.vim/ftplugin/chocodonut.vim`):

```
echo "Calling from chocodonut ftplugin"
```

创建另一个 ftplugin 文件，起名为`plaindonut.vim` (`~/.vim/ftplugin/plaindonut.vim`):

```
echo "Calling from plaindonut ftplugin"
```

现在，每次您打开一个 chocodonut 类型的文件时，Vim会运行 `~/.vim/ftplugin/chocodonut.vim`中的脚本。每次您打开 plaindonut 类型的文件时，Vim会运行 `~/.vim/ftplugin/plaindonut.vim` 中的脚本。

一个警告：每当一个buffer的文件类型被设置时(比如，`set filetype=chocodonut`)，上述脚本就会运行一次。如果您打开3个不同的 chocodonut 文件，该脚本将运行 *总共* 3次。

## 缩进文件

Vim有一个 缩进runtime路径，其工作方式与ftplugin类似，Vim也会在这个目录中查找一个与打开的文件类型名字一样的文件。缩进runtime路径的目的是存储缩进相关的代码。如果您有文件 `~/.vim/indent/chocodonut.vim`，它仅当您打开一个 chocodonut 类型的文件时执行。您可以将 chocodonut 文件中缩进相关的代码存储在这里。

## 颜色

Vim 有一个颜色runtime路径 (`~/.vim/colors/`) ，用来存储颜色主题。这个目录中的任何文件都会在命令行命令 `:color` 中显示出来。

如果您有一个文件 `~/.vim/colors/beautifulprettycolors.vim`，当您运行 `:color` 然后按 Tab，您将会看到 `beautifulprettycolors` 出现在颜色选项中。  如果您想添加自己的颜色主题，就放在这个地方。

如果您想看其他人做的颜色主题，有一个好地方值得推荐：[vimcolors](https://vimcolors.com/)。

## 语法高亮

Vim有一个语法runtime路径 (`~/.vim/syntax/`)，用来定义语法高亮。

假设您有一个文件 `hello.chocodonut`，在文件里面有以下内容：

```
(donut "tasty")
(donut "savory")
```

虽然Vim现在知道了正确的文件类型，但所有的文本都是相同的颜色。让我们添加语法高亮规则，使 "donut" 关键词高亮显示。创建一个新的 chocodonut 语法文件 `~/.vim/syntax/chocodonut.vim`，在文件中添加：

```
syntax keyword donutKeyword donut

highlight link donutKeyword Keyword
```

现在重新打开 `hello.chocodonut` 文件，关键词 `donut` 已经高亮显示了。

本章不会详细介绍语法高亮。它是一个庞大的主题。如果您感兴趣，可以查阅 `:h syntax.txt`。

[vim-polyglot](https://github.com/sheerun/vim-polyglot) 插件非常的棒，它提供了很多流行的编程语言的语法高亮。

## 文档

如果您写了一个插件，您还得创建一个您自己的文档。您可以使用文档runtime路径完成这个。

让我们为 chocodonut 和 plaindonut 关键字创建一个基本文档。创建文件 `donut.txt` (`~/.vim/doc/donut.txt`)。在文件中，添加一下内容：

```
*chocodonut* Delicious chocolate donut

*plaindonut* No choco goodness but still delicious nonetheless
```

如果您试着搜索 `chocodonut` 或 `plaindonut` (`:h chocodonut` 或 `:h plaindonut`)，您找不到任何东西。

首先，你需要运行 `:helptags`来创建新的帮助入口。运行 `:helptags ~/.vim/doc/`

现在，如果您运行 `:h chocodonut` 或 `:h plaindonut`，您将找到上面那些新的帮助入口。注意，现在文件是只读的，而且类型是 "help"。

## 延时加载脚本

到现在，本章您学到的所有runtime路径都是自动运行的。如果您想手动加载一个脚本，可使用 autoload runtime路径。

创建一个目录名为 autoload(`~/.vim/autoload/`)。在目录中，创建一个新文件，起名为 `tasty.vim` (`~/.vim/autoload/tasty.vim`)。在文件中：

```
echo "tasty.vim global"

function tasty#donut()
  echo "tasty#donut"
endfunction
```

注意，函数名是 `tasty#donut` 而不是 `donut()`。要想使用autoload功能，井号(`#`)是必须的。在使用autoload功能时，函数的命名惯例是：

```
function fileName#functionName()
  ...
endfunction
```

在本例中，文件名是 `tasty.vim`，而函数名是`donut`。

要调用一个函数，可以使用 `call` 命令。让我们call这个函数 `:call tasty#donut()`。

您第一次调用这个函数时，您应当会 *同时* 看到两条信息 ("tasty.vim global" 和 "tasty#donut") 。后面再调用 `tasty#donut` 函数，将只会显示 "testy#donut"。

当您在Vim中打开一个文件，不像前面说的runtime路径，autoload脚本不会被自动加载。仅当您显式地调用 `tasty#donut()`，Vim才会查找文件`tasty.vim`，然后加载文件中的内容，包括函数 `tasty#donut()`。有些函数会占用大量资源，但我们又不常用，这时候 Autoload runtime路径就是最佳的解决方案。 

您可以在autoload目录任意添加嵌套的目录。如果您有一个runtime路径 `~/.vim/autoload/one/two/three/tasty.vim`，您可以使用`:call one#two#three#tasty#donut()`来调用函数。

## After脚本

Vim有一个 after runtime路径 (`~/.vim/after/`) ，它的结构是 `~/.vim/`的镜像。在此目录中的任何脚本都会最后执行，所以开发者通常使用这个路径来重载脚本。

比如，如果您想重载 `plugin/chocolate.vim` 中的脚本，您可以创建`~/.vim/after/plugin/chocolate.vim`来放置重载脚本。Vim将会先运行 `~/.vim/plugin/chocolate.vim`， *然后运行* `~/.vim/after/plugin/chocolate.vim`

## $VIMRUNTIME

Vim有一个环境变量 `$VIMRUNTIME` 用来加载默认脚本和支持文件。您可以运行 `:e $VIMRUNTIME`查看。

它的结构应该看起来很熟悉。它包含的很多runtime路径都是我们本章前面学过的。

回想第22章，当您打开Vim时，它会在6个不同的位置查找vimrc文件。当时我说最后一个位置就是 `$VIMRUNTIME/default.vim`，如果Vim在前5个位置查找用户vimrc文件失败，就会使用`default.vim` 作为vimrc。

不知您是否尝试过，运行Vim是不加载比如vim-polyglot之类的语法插件，但您的文件依然有语法高亮?这是因为当Vim在runtime路径查找语法文件失败时，会从`$VIMRUNTIME` 的语法目录中查找语法文件。

查阅 `:h $VIMRUNTIME`了解更多信息。

## Runtimepath选项

运行 `:set runtimepath?`，可以查看您的runtime路径。

如果您使用 Vim-Plug 或其他流行的第三方插件管理器，它应该会显示一个目录列表。比如，我的显示如下：

```
runtimepath=~/.vim,~/.vim/plugged/vim-signify,~/.vim/plugged/base16-vim,~/.vim/plugged/fzf.vim,~/.vim/plugged/fzf,~/.vim/plugged/vim-gutentags,~/.vim/plugged/tcomment_vim,~/.vim/plugged/emmet-vim,~/.vim/plugged/vim-fugitive,~/.vim/plugged/vim-sensible,~/.vim/plugged/lightline.vim, ...
```

插件管理器做了一件事，就是将每个插件添加到runtime路径中。每个runtime路径都有一个类似 `~/.vim/`的目录结构。

如果您有一个目录 `~/box/of/donuts/`，然后您想将这个目录添加到您的runtime路径中，您可以在vimrc中添加以下内容：

```
set rtp+=$HOME/box/of/donuts/
```

如果在 `~/box/of/donuts/` 里面，您有一个plugin目录 (`~/box/of/donuts/plugin/hello.vim`) 以及ftplugin目录 (`~/box/of/donuts/ftplugin/chocodonut.vim`)，当您打开Vim时，Vim将会运行 `plugin/hello.vim` 中所有脚本。同样，当您打开一个 chocodonut 文件时，Vim 将会运行 `ftplugin/chocodonut.vim`。

自己试着做一下：创建一个任意目录，然后将它添加到您的 runtimepath中。添加一些我们本章学到的runtime路径。确保它们按预期工作。

## 聪明地学习Runtime

花点时间阅读本章，还有认真研究一下这些runtime路径。看一下真实环境下runtime路径是如何使用的。浏览一下您最喜欢的Vim插件仓库，仔细研究一下它的目录结构，您应该能够理解它们中的绝大部分。试着领会重点并跟着做。现在您已经理解了Vim的目录结构，您可以准备学习Vimscript了。

## 链接
- [目录](./directory.md)
- 上一部分 [Ch 23 - Vim软件包](./ch23_vim_packages.md)
