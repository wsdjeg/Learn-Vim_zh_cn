# 第3章 打开和搜索文件

本章的目的是向您介绍如何在Vim中快速搜索，能够快速搜索是提高您的Vim工作效率的重要途径。当我解决了如何快速搜索文件这个问题后，我就决定改为完全使用Vim来工作。

本章划分为两个部分：一是如何不依赖插件搜索；二是使用[fzf插件](https://github.com/junegunn/fzf.vim)搜索。让我们开始吧！

## 打开和编辑文件

要在Vim中打开一个文件，您可以使用`:edit`。

```
:edit file.txt
```

如果`file.txt`已经存在，就会打开`file.txt`buffer。如果`file.txt`不存在，会创建一个新buffer名为`file.txt`。

`:edit`命令支持使用`<Tab>`进行自动补全。比如，如果您的文件位于[Rails](https://rubyonrails.org/)应用控制器的用户控制器目录`./app/controllers/users_controllers.rb`内，您可以使用`<Tab>`对文件路径名进行快速扩展。

```
:edit a<Tab>c<Tab>u<Tab>
```

`:edit`可以接收通配符参数。`*`匹配当前目录下的任意文件。如果您只想查找当前目录下后缀名为`.yml`的文件：

```
:edit *.yml<Tab>
```

Vim将列出当前目录下所有`.yml`文件供您选择。

您可以使用`**`进行递归的搜索。如果您想查找当前项目文件夹下所有`*.md`文件，但您不知道在哪个目录，您可以这样做：

```
:edit **/*.md<Tab>
```

`:edit`可以用于运行`netrw`（Vim的内置文件浏览器）。使用方法是，给`:edit`一个目录参数而不是文件名就行了：

```
:edit .
:edit test/unit/
```

## 使用find命令搜索文件

您可以使用`:find`命令搜索文件。比如：

```
:find package.json
:find app/controllers/users_controller.rb
```

`:find`命令同样支持自动补全：

```
:find p<Tab>                " to find package.json
:find a<Tab>c<Tab>u<Tab>    " to find app/controllers/users_controller.rb
```

您可能注意到`:find`和`:edit`看起来很像。它们的区别是什么呢？

## Find 和 Path

两者的区别在于，`:find`命令根据`path`选项配置的路径查找文件，而`:edit`不会。让我们了解一点关于`path`选项的知识。一旦您学会如何修改您的路径，`:find`命令能变成一个功能强大的搜索工具。先查看一下您的`path`是什么：

```
:set path?
```

默认情况下，您的`path`内容很可能是这样的：

```
path=.,/usr/include,,
```

- `.` 意思是在当前文件所在目录下搜索。(译者注：注意不是命令行输入pwd返回的当前目录，而是 **当前所打开的文件** 所在的目录)
- `,` means to search in the current directory.(译者注：此处貌似作者有点小错误，逗号`,`应该是表示路径之间的分割符。连续的两个`,,`（两个逗号之间为空）才表示当前目录)
- `/usr/include` 表示在C编译器头文件目录下搜索。

前两个配置非常重要，第3个现在可以被忽略。您这里应该记住的是：您可以修改您自己的路径。让我们假设您的项目结构是这样的：

```
app/
  assets/
  controllers/
    application_controller.rb
    comments_controller.rb
    users_controller.rb
    ...
```

如果您想从根目录跳到`users_controller.rb`，您将不得不经过好几层目录（按好几次`<Tab>`）。一般说来，当您处理一个framework时，90%的时间您都在某个特定的目录下。在这种情况下，您只关心如何用最少的按键跳到`controllers/`目录。那么`path`设置可以减少这个途程。

您只需要将`app/controllers/`添加到当前`path`选项。以下是操作步骤：

```
:set path+=app/controllers/
```

现在您的路径已经更新，当您输入`:find u<Tab>`时，Vim将会在`app/controllers/`目录内搜索所有以"u"开头的文件。

如果您有一个嵌套的目录`controllers/`，比如`app/controllers/account/users_controller.rb`，Vim就找不到`users_controllers`了。您必须改为添加`:set path+=app/controllers/**`，这样自动补全才会找到`users_controller.rb`。这太棒了！您现在可以只需要按1次键就可找到users controller。

您可能会想将整个项目文件夹添加到`path`中，这样当您按`<Tab>`，Vim将在所有文件夹内搜索您要找的文件，就像这样：

```
:set path+=$PWD/**
```

`$PWD` 表示的是当前工作目录。如果您尝试将整个项目路径加到`path`中，并希望让所有文件名可以用`<Tab>`补全，虽然对于小项目没问题，但如果您的项目中包含大量文件时，这会显著降低您的搜索速度。我建议仅仅将您最常访问的文件/目录添加到`path`。（译者注：不知道是不是因为系统环境不一样的原因，译者使用的是windows下的Vim8.2，\$PWD 这个环境变量在译者的vim中不起作用，必须在vimrc文件中添加一句`let $PWD=getcwd()`才行）。

您可以将`set path+={您需要添加的目录}`添加到您的vimrc文件中。更新`path`仅花费几秒钟，但可以为您的工作节省很多时间。

## 使用Grep命令在文件中搜索

如果您想在文件内部查找（搜索文件中的词句），您可以使用grep。Vim有两个方法可以完成这个工作：

- 内置grep （`:vim`。没错，就是`:vim`，它是`:vimgrep`的简写）。
- 外部grep (`:grep`)。

让我们首先仔细看看内置grep。`:vim`有以下语法：

```
:vim /pattern/ file
```

- `/pattern/` 是您要搜索的内容的正则表达式。
- `file` 是文件参数。您可以传入多个参数。Vim将在文件中搜索所有匹配正则表达式的内容。类似于`:find`，您可以传入*和**通配符。

比如，要在`app/controllers/`目录下所有ruby文件(`.rb`)中，查找所有的"breakfast"字符串:

```
:vim /breakfast/ app/controllers/**/*.rb
```

输入上面的命令后，您将会被导航到第一个结果。Vim的`vim`搜索命令使用`quickfix`进行处理。要查看所有搜索结果，运行`:copen`会打开一个`quickfix`窗口。下面有一些有用的quickfix命令，可以让您提高效率：

```
:copen        打开quickfix窗口
:cclose       关闭quickfix窗口
:cnext        跳到下一个错误
:cprevious    跳到前一个错误
:colder       跳到旧的错误列表
:cnewer       跳到新的错误列表
```

要了解更多关于quickfix的信息，使用`:h quickfix`查看帮助信息。

您可能注意到运行内置grep(`:vim`)命令时，如果匹配结果数量较多时系统速度会变慢。这是因为Vim将每一个搜索结果读入内存。Vim加载每一个匹配的文件就像它们被编辑一样。如果Vim查到大量文件，它将消耗很多内存。

让我们谈谈外置grep。默认情况下，它使用终端命令`grep`。要想在`app/controllers/`目录中搜索字符串"lunch"，您可以这样做：

```
:grep -R "lunch" app/controllers/
```

注意这里不是使用`/pattern/`，它遵循的是终端grep的语法`"pattern"`，它同样使用'quickfix'来显示所有的匹配结果。

Vim使用`grepprg`变量来决定运行`:grep`时，应该使用哪个外部程序。所以您并不是非得使用终端的`grep`命令。稍后我将为您演示如何改变外部grep程序的默认值。

## 用Netrw浏览文件

`netrw`是Vim的内置文件浏览器，当查看项目的目录结构时很有用。要运行`netrw`，您需要在您的`.vimrc`中做以下设置：

```
set nocp
filetype plugin on
```

由于`netrw`是一个很宽泛的话题，我将仅仅介绍它的基本用法，这应该已经足够了。您可以在启动Vim时运行`netrw`，只需要传给Vim一个目录参数（而不是文件参数）就行了。比如：

```
vim .
vim src/client/
vim app/controllers/
```

要想从Vim内部运行`netrw`，您可以使用`:edit`命令，传给他一个目录而不是文件名：

```
:edit .
:edit src/client/
:edit app/controllers/
```

也有其他方法，不需要传递目录参数就运行`netrw`窗口：

```
:Explore     从当前文件启动netrw。
:Sexplore    Sex_Plore?不是开玩笑:)，在顶部水平分割的窗口打开netrw。
:Vexplore    在左侧垂直分割的窗口打开netrw。
```

您可以使用Vim动作(motions，在后面的章节中将详细讲述)在`netrw`中导航。如果您要创建、删除、重命名文件或文件夹，下面有一些关于`netrw`的有用命令：

```
%    创建新文件
d    创建新目录
R    重命名文件/目录
D    删除文件/目录
```

`:h netrw` 的信息非常复杂，如果您有时间可以看看。

如果您觉得 `netrw` 过于单调乏味，[vim-vinegar](https://github.com/tpope/vim-vinegar)是netrw的一个改进插件。如果您想找一个不同的文件浏览器，[NERDTree](https://github.com/preservim/nerdtree) 是一个很好的选择。去看看吧。

## Fzf插件

您现在已经学会了如何使用Vim的内置工具去搜索文件，那么让我们学习一下如何用插件实现这些功能吧。

现代文本编辑器相比Vim，有一个功能设计得很好，那就是文件搜索和模糊搜索的简化。在本章的下半部分，我将向您演示如何使用[fzf.vim](https://github.com/junegunn/fzf.vim)插件，在Vim中轻松实现功能强大的搜索功能。

## 安装

首先，确保您下载了[fzf](https://github.com/junegunn/fzf)和[ripgrep](https://github.com/BurntSushi/ripgrep)。按照它们在github仓库上的指令一步步做。成功安装后，命令`fzf`和`rg`应该就可以用了。

Ripgrep是一个类似grep（从名字上就看得出）的搜索工具。一般说来，它比grep要快，而且还有很多有用的特性。Fzf是一个多用途的命令行模糊搜索工具，您可以讲它与其他命令联合起来使用，包括ripgrep。联合起来后，它们成为一个功能强大的搜索工具。

Fzf默认并不使用ripgrep，所以我们需要设置`FZF_DEFAULT_COMMAND`变量告诉fzf使用ripgrep命令。在我的`.zshrc`（如果您用bash，应该是`.bashrc`）文件内，我有以下设置：

```
if type rg &> /dev/null; then
  export FZF_DEFAULT_COMMAND='rg --files'
  export FZF_DEFAULT_OPTS='-m'
fi
```

注意`FZF_DEFAULT_OPTS`变量中的`-m`。这个设置允许我们按下`<Tab`或`<Shift-Tab>`后进行多重选择。如果仅想让fzf在Vim中能够工作，这个设置不是必须的，但我认为这是一个有用的设置。当您想在多个文件中执行搜索和替换，进行少量修改时，它会很方便。fzf命令可以接收很多标志，但我不会再这里讲。要想了解更多信息，可以查看[fzf's 仓库](https://github.com/junegunn/fzf#usage)，或者使用`man fzf`。要想让fzf使用ripgrep，您至少得有这个设置`export FZF_DEFAULT_COMMAND='rg'`。

安装好了fzf和ripgrep后，让我们再安装fzf的插件。在这个例子中，我使用的是[vim-plug](https://github.com/junegunn/vim-plug)插件管理器，当然您可以使用其他插件管理器。

将下列配置添加到您的`.vimrc`中。因为您需要使用[fzf.vim](https://github.com/junegunn/fzf.vim)插件。（同样是由fzf的作者在维护）

```
Plug 'junegunn/fzf.vim'
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
```

添加后，您需要打开vim，运行`:PlugInstall`。这条命令将会安装所有您在`vimrc`文件中定义了但尚未安装的插件。 在我的例子中，将会安装`fzf.vim`和`fzf`。

要了解更多关于此插件的信息，您可以查看[fzf.vim 的仓库](https://github.com/junegunn/fzf/blob/master/README-VIM.md)。

## Fzf的语法

要想高效的使用fzf，您首先得了解一些fzf的基础语法。幸运的是，这个列表比较短：

- `^` 表示前缀精确匹配。要搜索一个以"welcome"开头的短语：`^welcom`。
- `$` 表示后缀精确匹配。要搜索一个以"my friends"结尾的短语：`friends$`。
- `'` 表示精确匹配。要搜索短语"welcom my friends"：`'welcom my friends`。
- `|` 表示"或者"匹配。要搜索"friends"或"foes"：`friends | foes`。
- `!` 表示反向匹配。要搜索一个包含"welcome"但不包含"friends"的短语：`welcome !friends`。

您可以混合起来使用。比如，`^hello | ^welcome friends$`将搜索以"welcome"或"hello"开头，并且以"friends"结束的短语。

## 查找文件

要想在Vim内使用fzf.vim插件搜索文件，您可以使用`:Files`方法。在Vim中运行`:Files`，您将看到fzf搜索提示符。

因为您将频繁地使用这个命令，最好建立一个键盘映射，我把它映射到`Ctrl-f`。在我的vimrc配置中，有这个设置：

```
nnoremap <silent> <C-f> :Files<CR>
```

## 在文件中查找

要想在文件内部搜索，您可以使用`:Rg`命令。

同样，因为您可能将频繁的使用这个命令，让我们给它一个键盘映射。我的映射在`<Leader>f`。

```
nnoremap <silent> <Leader>f :Rg<CR>
```

## 其他搜索

Fzf.vim提供了许多其他命令。这里我不会一个个仔细讲，您可以去[这里](https://github.com/junegunn/fzf.vim#commands)查看更多信息。

这是我的fzf键盘映射：

```
nnoremap <silent> <Leader>b :Buffers<CR>
nnoremap <silent> <C-f> :Files<CR>
nnoremap <silent> <Leader>f :Rg<CR>
nnoremap <silent> <Leader>/ :BLines<CR>
nnoremap <silent> <Leader>' :Marks<CR>
nnoremap <silent> <Leader>g :Commits<CR>
nnoremap <silent> <Leader>H :Helptags<CR>
nnoremap <silent> <Leader>hh :History<CR>
nnoremap <silent> <Leader>h: :History:<CR>
nnoremap <silent> <Leader>h/ :History/<CR>
```

## 将Grep替换为Rg

正如前面提到的，Vim有两种方法在文件内搜索：`:vim`和`:grep`。您可以使用`grepprg`这个关键字重新指定`:grep`使用的外部搜索工具。我将向您演示如何设置Vim，使得当运行`:grep`命令时，使用ripgrep代替终端的grep。

现在，让我们设置`grepprg`来使`:grep`使用ripgrep。将下列设置添加到您的vimrc：

```
set grepprg=rg\ --vimgrep\ --smart-case\ --follow
```

上面的一些选项可以随意修改！要想了解更多关于这些选项的含义，请使用`man rg`了解详情。

当您更新`grepprg`选项后，现在当您运行`:grep`，它将实际运行`rg --vimgrep --smart-case --follow`而不是`grep`。如果您想使用ripgrep搜索"donut"，您可以运行一条更简洁的命令`:grep "donut"`，而不是`:grep "donut" . -R`

就像老的`:grep`一样，新的`:grep`同样使用quickfix窗口来显示结果。

您可能好奇，“很好，但我从没在Vim中使用过`:grep`，为什么我不能直接使用`:Rg`命令在文件中搜索呢？究竟什么时候我必须使用`:grep`？”。

这个问题问得很好。在Vim中，当您需要在多个文件中执行搜索和替换时，您可能必须使用`:grep`这个命令。我马上就会讲这个问题。

## 在多文件中搜索和替换

现代文本编辑器，比如VSCode中，在多个文件中搜索和替换一个字符串是很简单的事情。在这一节，我将向您演示如何在Vim中轻松实现这个。

第一个方法是在您的项目中替换 **所有** 的匹配短句。您得使用`:grep`命令。如果您想将所有"pizza"替换为"donut"，下面是操作方法：

```
:grep "pizza"
:cfdo %s/pizza/donut/g | update
```

让我们来分析一下这条命令：

1. `:grep pizza`使用ripgrep去搜索所有"pizza"（顺带说一句，就算您不给`grepprg`重新赋值让它使用ripgrep，这条命令依然有效，但您可能不得不使用`:grep "pizza" . -R`命令，而不是`:grep "pizza"`）。
2. `:cfdo`会在您的quickfix列表中所有文件里，执行您传递给它的命令。在这个例子中，您的命令是一条替换命令`%s/pizza/donut/g`。管道符号(`|`)是一个链接操作符。命令`update`在每个文件被替换后，立刻保存。在后面的章节中，我将深入介绍替换命令。

第二个方法是在您选择文件中执行搜索和替换。用这个方法，您可以手动选择您想执行搜索和替换的文件。下面是操作方法：

1. 首先清空您的buffer。让您的buffer列表仅包含您所需要的文件，这一点很有必要。您可以重启Vim，也可以运行`:%bd | e#`命令（`%bd`关闭所有buffer，而`e#`打开您当前所在的文件）。
2. 运行`:Files`。
3. 选择好您想搜索-替换的文件。要选择多个文件，使用`<Tab>`或`<Shift-Tab>`。当然，您必须使多文件标志(`-m`)位于`FZF_DEFAULT_OPTS`中。
4. 运行`:bufdo %s/pizza/donut/g | update`。命令`:bufdo %s/pizza/donut/g | update`看起来和前面的`:cfdo %s/pizza/donut/g | update`很像，区别在于，(`:cfdo`)替换所有quickfix中的实体，而(`:bufdo`)替换所有buffer中的实体。

## 用聪明的方法学习搜索

在文本编辑时，搜索是一个很实用的技巧。学会在Vim中如何搜索，将显著提高您的文本编辑工作流程效率。

Fzf.vim插件就像一个游戏规则改变者。我无法想象使用Vim没有它的情景。当最开始使用Vim时，如果有一个好的搜索工具，我想是非常重要的。我看见很多人过渡到Vim时的艰难历程，就是因为Vim缺少了现代编辑器所拥有的一些关键功能特性，比如简单快捷且功能强大的搜索功能。我希望本章将帮助您更轻松地向Vim过渡。

您同时也看到了Vim的扩展性，即使用插件或外部程序扩展搜索功能的能力。将来，记住您想在Vim中拓展的功能。很有可能已经有人写好了相关插件，已经有现成的程序了。下一章，您将学习Vim中非常重要的主题：Vim语法。

## 链接
- [目录](./directory.md)
- 上一部分 [Ch 2 - 缓冲区，窗口和选项卡](./ch02_buffers_windows_tabs.md)
- 下一部分 [Ch 4 - Vim 语法](./ch04_vim_grammar.md)
