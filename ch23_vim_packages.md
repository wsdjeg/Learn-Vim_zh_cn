# 第23章 Vim软件包

在前面的章节中，我提到使用第三方插件管理器来安装插件。从Vim 8开始，Vim自带了一个内置的插件管理器，名叫 *软件包（packages）*。在本章，您将学习如何使用Vim软件包来安装插件。

要看您的Vim编译版本是否能够使用软件包，运行 `:version`。然后查看是否有 `+packages`属性。另外，您也可以运行 `:echo has('packages')`（如果返回1，表示可以使用软件包）。

## 包目录

在根目录下查看您是否有一个 `~/.vim` 文件夹。如果没有就新建一个。在文件夹里面，创建一个子文件夹取名 `pack`(`~/.vim/pack/`)。Vim会在这个子文件夹内自动搜索插件。

## 两种加载方式

Vim软件包有两种加载机制：自动加载和手动加载。

### 自动加载

要想让Vim启动时自动加载插件，你需要将它们放置在 `start/`子目录中。路径看起来像这样：

```
~/.vim/pack/*/start/
```

现在您可能会问，为什么在`pack/` 和 `start/` 之间有一个 `*` ？这个星号可以是任意名字。让我们将它取为`packdemo/`：

```
~/.vim/pack/packdemo/start/
```

记住，如果您忽略这一点，用下面的路径代替的话：

```
~/.vim/pack/start/
```

软件包系统是不会正常工作的。 必须在`pack/` 和 `start/`之间添加一个名字才能正常运行。

在这个示例中，让我们尝试安装 [NERDTree](https://github.com/preservim/nThe package system won't work. It is imperative to put a name between `pack/` and `start/`.erdtree) 插件。用任意方法进入 `start/` 目录（`cd ~/.vim/pack/packdemo/start/`），然后将NERDTree的仓库克隆下来：

```
git clone https://github.com/preservim/nerdtree.git
```

完成了！您已经完成了安装。下一次您启动Vim，您可以立即执行 NERDTree 命令 `:NERDTreeToggle`。

在 `~/.vim/pack/*/start/` 目录中，您想克隆多少插件仓库就克隆多少。Vim将会自动加载每一个插件。如果您删除了克隆的仓库（`rm -rf nerdtree`），那么插件就失效了。

### 手动加载

要想在Vim启动时手动加载插件，您得将相关插件放置在 `opt/` 目录中，类似于自动加载，这个路径看起来像这样：

```
~/.vim/pack/*/opt/
```

让我们继续使用前面的 `packdemo/` 这个名字：

```
~/.vim/pack/packdemo/opt/
```

这一次，让我们安装[killersheep](https://github.com/vim/killersheep) 游戏（需要Vim8.2以上版本）。进入`opt/` 目录(`cd ~/.vim/pack/packdemo/opt/`) 然后克隆仓库：

```
git clone https://github.com/vim/killersheep.git
```

启动Vim。执行游戏的命令是 `:KillKillKill`。试着运行一下。Vim将会提示这不是一个有效的编辑命令。您需要首先 *手动* 加载插件，运行：

```
:packadd killersheep
```

现在再运行一下 `:KillKillKill` 。命令已经可以使用了。

您可能好奇，“为什么我需要手动加载插件？启动时自动加载岂不是更好？”

很好的问题。有时候有些插件我们并不是所有的时候都在用，比如 KillerSheep 游戏。您可能不会想要加载10个不同的游戏导致Vim启动变慢。但是偶尔当您觉得乏味的时候，您可能想要玩几个游戏，使用手动加载一些非必须的插件。

您也可以使用这个方法有条件的加载插件。可能您同时使用了Neovim和Vim，有一些插件是为NeoVim优化过的。您可以添加类似下列的内容到您的vimrc中：

```
if has('nvim')
  packadd! neovim-only-plugin
else
  packadd! generic-vim-plugin
endif
```

## 组织管理软件包

回想一下，要使用Vim的软件包系统必须有以下需求：

```
~/.vim/pack/*/start/
```

或者:

```
~/.vim/pack/*/opt/
```

实际上，`*`星号可以使 *任意* 名字，这个名字就可以用来管理您的插件。假设您想将您的插件根据类型（颜色、语法、游戏）分组：

```
~/.vim/pack/colors/
~/.vim/pack/syntax/
~/.vim/pack/games/
```

您仍然可以使用各个目录下的 `start/` 和 `opt/` 。

```
~/.vim/pack/colors/start/
~/.vim/pack/colors/opt/

~/.vim/pack/syntax/start/
~/.vim/pack/syntax/opt/

~/.vim/pack/games/start/
~/.vim/pack/games/opt/
```

## 聪明地添加插件

您可能好奇，Vim软件包是否让一些流行的插件管理器，比如 vim-pathogen, vundle.vim, dein.vim, a还有vim-plug面临淘汰？

答案永远是：“看情况而定。”

我仍然使用vim-plug，因为使用它添加、删除、更新插件很容易。如果您使用了很多插件，插件管理器的好处更加明显，因为使用它可以对很多插件进行同时更新。有些插件管理器同时也提供了一些异步功能。

如果您是极简主义者，可以尝试一下Vim软件包。如果您是一名插件重度使用者，您可能需要一个插件管理器。

## 链接
- [目录](./directory.md)
- 上一部分 [Ch 22 - Vimrc](./ch22_vimrc.md)
- 下一部分 [Ch 24 - Vim Runtime](./ch24_vim_runtime.md)
