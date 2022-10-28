# 第22章 Vimrc

在先前的章节中，您学习了如何使用Vim。在本章，您将学习如何组织和配置Vimrc。

## Vim如何找到Vimrc

对于Vimrc，常见的理解是在根目录下添加一个 `.vimrc` 点文件（根据您使用的操作系统，文件路径名可能不同）。

实际上，Vim在多个地方查找vimrc文件。下面是Vim检查的路径：
- `$VIMINIT`
- `$HOME/.vimrc`
- `$HOME/.vim/vimrc`
- `$EXINIT`
- `$HOME/.exrc`
- `$VIMRUNTIME/default.vim`

当您启动Vim时，它将在上面列出的6个位置按顺序检查vimrc文件，第一个被找到的vimrc文件将被加载，而其余的将被忽略。

首先，Vim将查找环境变量 `$VIMINIT`。如果没有找到，Vim将检查 `$HOME/.vimrc`。如果还没找到，VIm就检查 `$HOME/.vim/vimrc`。如果Vim找到了vimrc文件，它就停止查找，并使用 `$HOME/.vim/vimrc`。

关于第一个位置，`$VIMINIT` 是一个环境变量。默认情况下它是未定义的。如果您想将 `~/dotfiles/testvimrc` 作为 `$VIMINTI` 的值，您可以创建一个包含那个vimrc路径的环境变量。当您运行 `export VIMINIT='let $MYVIMRC="$HOME/dotfiles/testvimrc" | source $MYVIMRC'`后，VIm将使用 `~/dotfiles/testvimrc` 作为您的vimrc文件。

第二个位置，`$HOME/.vimrc` 是很多Vim用户习惯使用的路径。`$HOME` 大部分情况下是您的根目录（`~`）。如果您有一个 `~/.vimrc` 文件，Vim将使用它作为您的vimrc文件。

第三个，`$HOME/.vim/vimrc`，位于 `~/.vim` 目录中。您可能已经有了一个 `~/.vim` 目录用于存放插件、自定义脚本、或视图文件。注意这里的vimrc文件名没有“点”（`$HOME/.vim/.vimrc` 不会被识别，但 `$HOME/.vim/vimrc`能被识别）。

第四个，`$EXINIT` 工作方式与 `$VIMINIT` 类似。

第五个，`$HOME/.exrc` 工作方式与 `$HOME/.vimrc` 类似。

第六个，`$VIMRUNTIME/defaults.vim` 是Vim编译时自带的默认vimrc文件。在我的电脑中，我是使用Homebrew安装的Vim8.2，所以我的路径是（`/usr/local/share/vim/vim82`）。如果Vim在前5个位置都没有找到vimrc文件，它将使用这个Vim自带的vimrc文件。

在本章剩余部分，我将假设vimrc使用的路径是 `~/.vimrc`。

## 应该把什么放在Vimrc中？

我刚开始配置Vimrc时，曾问过一个问题，“我究竟该把什么放在Vimrc文件中？”。

答案是，“任何您想放的东西”。 直接复制粘贴别人的vimrc文件的确是一个诱惑，但您应当抵制这个诱惑。如果您仍然坚持使用别人的vimrc文件，确保您知道这个vimrc干了什么，为什么他/她要用这些设置？以及他/她如何使用这些设置？还有最重要的是，这个vimrc文件是否符合你的实际需要？别人使用并不代表您也要使用。

## Vimrc基础内容

简单地说，一个vimrc是以下内容的集合：
- 插件
- 设置
- 自定义函数
- 自定义命令
- 键盘映射

当然还有一些上面没有提到的内容，但总体说，已经涵盖了绝大部分使用场景。

### 插件

在前面的章节中，我曾提到很多不同的插件，比如[fzf.vim](https://github.com/junegunn/fzf.vim), [vim-mundo](https://github.com/simnalamburt/vim-mundo), 还有 [vim-fugitive](https://github.com/tpope/vim-fugitive).

十年前，管理插件插件是一个噩梦。但随着很多现代插件管理器的开发，现在安装插件可以在几秒内完成。我现在正在使用[vim-plug](https://github.com/junegunn/vim-plug)作为我的插件管理器，所以我在本节中将使用它。相关概念和其他流行的插件管理器应该是类似的。我强烈建议您多试试几个插件管理器，比如：
- [vundle.vim](https://github.com/VundleVim/Vundle.vim)
- [vim-pathogen](https://github.com/tpope/vim-pathogen)
- [dein.vim](https://github.com/Shougo/dein.vim)

除了上面列出的，还有很多插件管理器，可以随便看看。要想安装 vim-plug，如果您使用的是Unix，运行：

```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

要添加新的插件，将您的插件名(比如，`Plug 'github-username/repository-name'`) 放置在 `call plug#begin()` 和 `call plug#end()` 之间的行中. 所以，如果您想安装 `emmet-vim` 和 `nerdtree`，将下面的片段放到您的vimrc中：

```
call plug#begin('~/.vim/plugged')
  Plug 'mattn/emmet-vim'
  Plug 'preservim/nerdtree'
call plug#end()
```

然后保存修改，加载当前vimrc (`:source %`), 然后运行 `:PlugInstall` 安装插件。

如果以后您想删除不使用的插件，您只需将插件名从 `call` 代码块之间移除，保存并加载，然后运行 `:PlugClean` 命令将它从机器上删除。

Vim 8 有自己的内置包管理器。您可以查阅 `:h packages` 了解更多信息。在后面一章中，我将向您展示如何使用它。

### 设置

在任意一个vimrc文件中都可以看到大量的 `set` 选项。 如果您在命令行模式中运行 set 命令，它只是暂时的。当您关闭Vim，设置就会丢失。比如，为了避免您每次运行Vim时都必须在命令行模式运行 `:set relativenumber number` 命令，您可以将这个命令添加在vimrc中：

```
set relativenumber number
```

有一些设置需要您赋予一个值，比如 `set tabstop=2`。想了解一个设置可以接收什么类型的值，可以查看帮助页。

您也可以使用 `let` 来代替 `set`（确保在选项前添加一个 `&`号）。使用 `let` ，您可以使用表达式进行赋值。比如，要想仅当某个路径存在时，才将该路径赋予 `'dictionary'` 选项：

```
let s:english_dict = "/usr/share/dict/words"

if filereadable(s:english_dict)
  let &dictionary=s:english_dict
endif
```

在后面的章节中您将了解关于Vimscript赋值和条件的知识。

要查看Vim中所有可用的选项，查阅 `:h E355`。

### 自定义函数

Vimrc是一个很好的用来放置自定义函数的地方。在后面的章节中您将学习如何写您自己的Vimscript函数。

### 自定义命令

您可以使用 `command` 创建一个自定义命令行命令。

比如，创建一个用于显示今天日期的基本命令 `GimmeDate`：

```
:command! GimmeDate echo call("strftime", ["%F"])
```

当您运行 `:GimmeDate` 时，Vim将显示一个类似 "2021-01-1"的日期。

要创建一个可以接收输入的基本命令，您可以使用 `<args>` 。如果您想向 `GimmeDate` 传递一个时间/日期格式参数：

```
:command! GimmeDate echo call("strftime", [<args>])

:GimmeDate "%F"
" 2020-01-01

:GimmeDate "%H:%M"
" 11:30
```

如果您想限定参数的数目，您可以使用 `-nargs` 标志。`-nargs=0` 表示没有参数，`-nargs=1` 表示传递1个参数，`-nargs=+` 表示至少1个参数，`-nargs=*` 表示传递任意数量的参数，`-nargs=?` 表示传递0个或1个参数。如果您想传递n个参数，使用 `-nargs=n`（这里 `n` 是一个任意整数）。

`<args>` 有两个变体：`<f-args>` 和 `<q-args>` 。前者用来向Vimscript函数传递参数，后者用来将用户输入自动转换为字符串。

使用 `args`:

```
:command! -nargs=1 Hello echo "Hello " . <args>
:Hello "Iggy"
" returns 'Hello Iggy'

:Hello Iggy
" Undefined variable error
```

使用 `q-args`:

```
:command! -nargs=1 Hello echo "Hello " . <q-args>
:Hello Iggy
" returns 'Hello Iggy'
```

使用 `f-args`:

```
:function! PrintHello(person1, person2)
:  echo "Hello " . a:person1 . " and " . a:person2
:endfunction

:command! -nargs=* Hello call PrintHello(<f-args>)

:Hello Iggy1 Iggy2
" returns "Hello Iggy1 and Iggy2"
```

当您学了关于Vimscript函数的章节后，上面的函数将更有意义。

查阅 `:h command` 和 `:args` 了解更多关于command和args的信息。

### 键盘映射

如果您发现您重复地执行一些相同的复杂操作，那么为这些复杂操作建立一个键盘映射将会很有用：

比如，在我的vimrc文件中有2个键盘映射：

```
nnoremap <silent> <C-f> :GFiles<CR>

nnoremap <Leader>tn :call ToggleNumber()<CR>
```

在第一个中，我将 `Ctrl-F` 映射到 [fzf.vim](https://github.com/junegunn/fzf.vim) 插件的 `:Gfiles` 命令(快速搜索Git文件)上。在第二个中，我将 `<leader>tn` 映射到调用一个自定义函数 `ToggleNumber` （切换 `norelativenumber` 和 `relativenumber` 选项）。`Ctrl-f` 映射覆盖了Vim的原生的页面滚动。如果发生冲突，您的映射将会覆盖Vim的设置。因为从几乎从来不用Vim原生的页面滚动功能，所以我认为可以安全地覆盖它。

另外，在 `<Leader>tn` 中的 "leader" 键到底是什么?

Vim有一个leader键用来辅助键盘映射。比如，我将 `<leader>tn` 映射为运行 `ToggleNumber()` 函数。如果没有leader键，我可能会用 `tn`，但Vim中的 `t` 已经用做其他功能（"till"搜索导航命令）了。有了leader键，我现在先按定义好的leader键作为开头，然后按 `tn`，而不用干扰已经存在的命令。您可以设置leader键作为您映射的连续按键的第一个按键。默认Vim使用反斜杠作为leader键（所以 `<Leader>tn` 会变成 "反斜杠-t-n"）。

我个人喜欢使用空格 `<Space>` 作为leader键，代替默认的反斜杠。要想改变您的leader键，将下面的文本添加到您的vimrc中：

```
let mapleader = "\<space>"
```

上面的 `nnoremap` 命令可以分解为三个部分：
- `n` 表示普通模式。
- `nore` 表示禁止递归。
- `map` 是键盘映射命令。

如果不想使用 `nnoremap`，您至少也得使用 `nmap` (`nmap <silent> <C-f> :Gfiles<CR>`)。但是，最好还是使用禁止递归的版本，这样是为了避免键盘映射时潜在的无限循环风险。

如果您进行键盘映射时不使用禁止递归，下面例子演示了会发生什么。假设您想给 `B` 添加一个键盘映射，用来在一行的末尾添加一个分号，然后跳回前一个词组（回想一下，`B` 是Vim普通模式的一个导航命令，用来跳回前一个词组)。 

```
nmap B A;<esc>B
```

当您按下 `B` ...哦豁，Vim开始失控了，开始无止尽的添加`;`（用 `Ctrl-c`终止）。为什么会发生这样的情况？因为在键盘映射 `A;<esc>B`中，这个 `B`不再是Vim原生的导航命令，它已经被映射到您刚才创建的键盘映射中了。这是您实际上执行的操作序列：

```
A;<esc>A;<esc>A;<esc>A;esc>...
```

要解决这个问题，您需要指定键盘映射禁止递归：

```
nnoremap B A;<esc>B
```

现在再按一下 `B` 试试。这一次它成功地在行尾添加了一个 `;`，然后跳回到前一个词组。这个映射中的 `B` 就表示Vim原生的 `B`了。

Vim针对不同的模式有不同的键盘映射命令。如果您想创建一个插入模式下的键盘映射 `jk`，用来退出插入模式：

```
inoremap jk <esc>
```

其他模式的键盘映射命令有：`map`（普通、可视、选择、以及操作符等待模式）， `vmap`（可视、选择）， `smap`（选择）， `xmap`（可视）， `omap`（操作符等待模式）， `map!`（插入、命令行）， `lmap`（插入，命令行，Lang-arg模式）， `cmap`（命令行）， 还有`tmap`（终端任务）。在这里我不会详细的讲解它们，要了解更多信息，查阅 `:h map.txt`。

创建最直观、最一致、最易于记忆的键盘映射。

## 组织管理Vimrc

一段时候键，您的vimrc文件就会变大且复杂得难以阅读。有两种方法让您的vimrc文件保持整洁：
- 将您的vimrc文件划分为几个文件
- 折叠您的vimrc文件

### 划分您的vimrc

您可以使用Vim的 `:source` 命令将您的vimrc文件划分为多个文件。这个命令可以根据给定的文件参数，读取文件中的命令行命令。

让我们在 `~/.vim` 下创建一个子文件夹，取名为 `/settings`（`~/.vim/settings`）。名字可以取为任意您喜欢的名字。

然后你在这个文件夹下创建4个文件：
- 第三方插件 (`~/.vim/settings/plugins.vim`).
- 通用设置 (`~/.vim/settings/configs.vim`).
- 自定义函数 (`~/.vim/settings/functions.vim`).
- 键盘映射 (`~/.vim/settings/mappings.vim`) .

在 `~/.vimrc` 里面添加:

```
source $HOME/.vim/settings/plugins.vim
source $HOME/.vim/settings/configs.vim
source $HOME/.vim/settings/functions.vim
source $HOME/.vim/settings/mappings.vim
```

在 `~/.vim/settings/plugins.vim` 里面:

```
call plug#begin('~/.vim/plugged')
  Plug 'mattn/emmet-vim'
  Plug 'preservim/nerdtree'
call plug#end()
```

在 `~/.vim/settings/configs.vim` 里面:

```
set nocompatible
set relativenumber
set number
```

在 `~/.vim/settings/functions.vim` 里面:

```
function! ToggleNumber()
  if(&relativenumber == 1)
    set norelativenumber
  else
    set relativenumber
  endif
endfunc
```

在 `~/.vim/settings/mappings.vim` 里面:

```
inoremap jk <esc>
nnoremap <silent> <C-f> :GFiles<CR>
nnoremap <Leader>tn :call ToggleNumber()<CR>
```

这样您的vimrc文件依然能够正常工作，但现在它只有4行了。

使用这样的设置，您可以轻易知道到哪去修改配置。如果您要添加一些键盘映射，就将它们添加在 `/mappings.vim` 文件中。以后，当您的vimrc变大时，您总是可以新建几个子文件来缩小它的大小。比如，如果您想为主题配色创建相关设置，您可以添加 `~/.vim/settings/themes.vim`。

### 保持单独的一个Vimrc文件

如果您倾向于保持一个单独的vimrc文件，以使它更加便于携带，您可以使用标志折叠让它保持有序。在vimrc文件的顶部添加一下内容：

```
" setup folds {{{
augroup filetype_vim
  autocmd!
  autocmd FileType vim setlocal foldmethod=marker
augroup END
" }}}
```

Vim能够检测当前buffer所属的文件类型 (`:set filetype?`). 如果发现属于 `vim` 类型，您可以使用标志折叠。回想一个标志折叠的用法，它使用 `{{{` 和 `}}}` 来指明折叠的开始和结束。

添加 `{{{` 和 `}}}` 标志将您的vimrc文件其他部分折叠起来。(别忘了使用 `"` 对标志进行注释):

```
" setup folds {{{
augroup filetype_vim
  autocmd!
  autocmd FileType vim setlocal foldmethod=marker
augroup END
" }}}

" plugins {{{
call plug#begin('~/.vim/plugged')
  Plug 'mattn/emmet-vim'
  Plug 'preservim/nerdtree'
call plug#end()
" }}}

" configs {{{
set nocompatible
set relativenumber
set number
" }}}

" functions {{{
function! ToggleNumber()
  if(&relativenumber == 1)
    set norelativenumber
  else
    set relativenumber
  endif
endfunc
" }}}

" mappings {{{
inoremap jk <esc>
nnoremap <silent> <C-f> :GFiles<CR>
nnoremap <Leader>tn :call ToggleNumber()<CR>
" }}}
```

您的vimrc文件将会看起来类似下面：

```
+-- 6 lines: setup folds -----

+-- 6 lines: plugins ---------

+-- 5 lines: configs ---------

+-- 9 lines: functions -------

+-- 5 lines: mappings --------
```

## 启动Vim时加载/不加载Vimrc和插件

如果您要启动Vim时，既不加载Vimrc，也不加载插件，运行：

```
vim -u NONE
```

如果您要启动Vim时，不加载Vimrc，但加载插件，运行：

```
vim -u NORC
```

如果您要启动Vim时，加载Vimrc，但不加载插件，运行

```
vim --noplugin
```

如果您要Vim启动加载一个 *其他的* vimrc, 比如 `~/.vimrc-backup`, 运行:

```
vim -u ~/.vimrc-backup
```

## 聪明地配置Vimrc

Vimrc是定制Vim时的一个重要组件，学习构建您的Vimrc最好是首先阅读他人的vimrc文件，然后逐渐地建立自己的。最好的vimrc并不是谁谁谁使用的，而是最适合您的工作需要和编辑风格的。

## 链接
- [目录](./directory.md)
- 上一部分 [Ch 21 - 多文件操作](./ch21_multiple_file_operations.md)
- 下一部分 [Ch 23 - Vim软件包](./ch23_vim_packages.md)