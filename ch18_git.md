# 第18章 Git

Vim 和 Git 是两种实现不同功能的伟大工具。Vim 用于文本编辑，Git 用于版本控制。

在本章中，您将学习如何将 Vim 和 Git 集成在一起。

## 差异比较

在上一章中，您看到了如何运行 `vimdiff` 命令以显示多个文件之间的差异。

假设您有两个文件，`file1.txt` 和 `file2.txt`。

`file1.txt` 的内容如下：

```
pancakes
waffles
apples

milk
apple juice

yogurt
```

`file2.txt` 的内容如下：

```
pancakes
waffles
oranges

milk
orange juice

yogurt
```

若要查看两个文件之间的差异，请运行：

```
vimdiff file1.txt file2.txt
```

或者也可以运行：

```
vim -d file1.txt file2.txt
```

<p align="center">
  <img alt="Basic diffing with Vim" width="900" height="auto" src="images/diffing-basic.png">
</p>

`vimdiff` 并排显示两个缓冲区。左边是 `file1.txt`，右边是 `file2.txt`。不同的两行（apples 和 oranges）会被高亮显示。

假设您要使第二个缓冲区相应位置变成 apples，而不是 oranges。若想从 `file1.txt` 传输您当前位置（当前您在 `file1.txt`）的内容到 `file2.txt`，首先使用 `]c` 跳转到下一处差异（使用 `[c` 可跳回上一处差异），现在光标应该在 apples 上了。接着运行 `:diffput`。此时，这两个文件都是 apples 了。

<p align="center">
  <img alt="Finding files in FZF" width="900" height="auto" src="images/diffing-apples.png">
</p>

如果您想从另一个缓冲区（orange juice，`file2.txt`）传输文本来替代当前缓冲区（apple juice，`file1.txt`），让您的光标仍然位于 `file1.txt` 的窗口中，首先使用 `]c` 跳转至下一处差异，此时光标应该在 apple juice 上。接着运行 `:diffget` 获取另一个缓冲区的 orange juice 来替代当前缓冲区中的 apple juice。

`:diffput` 将文本从当前缓冲区 *输出* 到另一个缓冲区。`:diffget` 从另一个缓冲区 *获取* 文本到当前缓冲区。

如果有多个缓冲区，可以运行 `:diffput fileN.txt` 和 `:diffget fileN.txt` 来指定目标缓冲区 fileN。

## 使用 Vim 作为合并工具

> “我非常喜欢解决合并冲突。” ——佚名

我不知道有谁喜欢解决合并冲突，但总之，合并冲突是无法避免的。在本节中，您将学习如何利用 Vim 作为解决合并冲突的工具。

首先，运行下列命令来将默认合并工具更改为 `vimdiff`：

```
git config merge.tool vimdiff
git config merge.conflictstyle diff3
git config mergetool.prompt false
```

或者您也可以直接修改 `~/.gitconfig`（默认情况下，它应该处于根目录中，但您的可能在不同的位置）。上面的命令应该会将您的 `gitconfig` 改成如下设置的样子，如果您还没有运行上面的命令，您也可以手动更改您的 gitconfig。

```
[core]
  editor = vim
[merge]
  tool = vimdiff
  conflictstyle = diff3
[difftool]
  prompt = false
```

让我们创建一个假的合并冲突来测试一下。首先创建一个目录 `/food`，并初始化 git 仓库：

```
git init
```

添加 `breakfast.txt` 文件，内容为：

```
pancakes
waffles
oranges
```

添加文件并提交它：

```
git add .
git commit -m "Initial breakfast commit"
```

接着，创建一个新分支 apples：

```
git checkout -b apples
```

更改 `breakfast.txt` 文件为：

```
pancakes
waffles
apples
```

保存文件，添加并提交更改：

```
git add .
git commit -m "Apples not oranges"
```

真棒！现在 master 分支有 oranges，而 apples 分支有 apples。接着回到 master 分支：

```
git checkout master
```

在 `breakfast.txt` 文件中，您应该能看到原来的文本 oranges。接着将它改成 grapes，因为它是现在的应季水果：

```
pancakes
waffles
grapes
```

保存、添加、提交：

```
git add .
git commit -m "Grapes not oranges"
```

嚯！这么多步骤！现在准备要将 apples 分支合并进 master 分支了：

```
git merge apples
```

您应该会看到如下错误：

```
Auto-merging breakfast.txt
CONFLICT (content): Merge conflict in breakfast.txt
Automatic merge failed; fix conflicts and then commit the result.
```

没错，一个冲突！现在一起来用一下新配置的 `mergetool` 来解决冲突吧！运行：

```
git mergetool
```

<p align="center">
  <img alt="Three-way mergetool with Vim" width="900" height="auto" src="images/mergetool-initial.png">
</p>

Vim 显示了四个窗口。注意一下顶部三个：

- `LOCAL` 包含了 `grapes`。这是“本地”中的变化，也是您要合并的内容。
- `BASE` 包含了 `oranges`。这是 `LOCAL` 和 `REMOTE` 的共同祖先，用于比较它们之间的分歧。
- `REMOTE` 包含了 `apples`。这是要被合并的内容。

底部窗口（也即第四个窗口），您能看到：

```
pancakes
waffles
<<<<<<< HEAD
grapes
||||||| db63958
oranges
=======
apples
>>>>>>> apples
```

第四个窗口包含了合并冲突文本。有了这步设置，就能更轻松看到哪个环境发生了什么变化。您可以同时查看 `LOCAL`、`BASE` 和 `REMOTE` 的内容。

您的光标应该在第四个窗口的高亮区域。再运行 `:diffget LOCAL`，就可以*获取*来自 `LOCAL` 的改变（grapes）。同样，运行 `:diffget BASE` 可以获取来自 `BASE` 的改变（oranges），而运行 `:diffget REMOTE` 可以获取来自 `REMOTE` 的改变（apples）。

在这个例子中，我们试着获取来自 `LOCAL` 的改变。运行 `:diffget LO`（`LOCAL` 的简写），第四个窗口变成了 grapes。完成后，就可以保存并退出所有文件（`:wqall`）了。还不错吧？

稍加留意您会发现，现在多了一个 `breakfast.txt.orig` 文件。这是 Git 防止事与愿违而创建的备份文件。如果您不希望 Git 在合并期间创建备份文件，可以运行：

```
git config --global mergetool.keepBackup false
```

## 在 Vim 中使用 Git

Vim 本身没有集成 Git，要在 Vim 中执行 Git 命令，一种方法是在命令行模式中使用 `!` 叹号运算符。

使用 `!` 可以运行任何 Git 命令：

```
:!git status
:!git commit
:!git diff
:!git push origin master
```

您还可以使用 Vim 的特殊字符 `%` (当前缓冲区) 或 `#` (其他缓冲区)：

```
:!git add %         " git add current file
:!git checkout #    " git checkout the other file
```

这里有一个Vim技巧，您可以用来添加不同Vim窗口中的多个文件，运行：

```
windo !git add %
```

然后提交：

```
:!git commit "添加了Vim窗口中的所有文件，酷"
```

`windo`命令是VIm的 "do" 命令其中之一，类似于您前面看到的 `argdo` 。`windo` 将命令执行在每一个窗口中。

## 插件

这里有很多提供git支持的Vim插件。以下是 Vim 中较流行的 Git 相关插件列表（您读到这篇文章时可能又有更多）：

- [vim-gitgutter](https://github.com/airblade/vim-gitgutter)
- [vim-signify](https://github.com/mhinz/vim-signify)
- [vim-fugitive](https://github.com/tpope/vim-fugitive)
- [gv.vim](https://github.com/junegunn/gv.vim)
- [vimagit](https://github.com/jreybert/vimagit)
- [vim-twiggy](https://github.com/sodapopcan/vim-twiggy)
- [rhubarb](https://github.com/tpope/vim-rhubarb)

其中最流行的是 vim-fugitive。本章的剩余部分，我将使用此插件来介绍几个 git 工作流。

## Vim-Fugitive

vim-fugitive 插件允许您在不离开 Vim 编辑器的情况下运行 git 命令行界面。您会发现，有些命令在 Vim 内部执行时会更好。

开始前，请先使用 Vim 插件管理器（[vim-plug](https://github.com/junegunn/vim-plug)、[vundle](https://github.com/VundleVim/Vundle.vim)、[dein.vim](https://github.com/Shougo/dein.vim) 等）安装 vim-fugitive。

## Git Status

当您不带参数地运行 `:Git` 命令时，vim-fugitive 将显示一个 git 概要窗口，它显示了未跟踪、未暂存和已暂存的文件。在此 “`git status`” 模式下，您可以做一些操作：

- `Ctrl-n` / `Ctrl-p` 转到下一个 / 上一个文件。
- `-` 暂存或取消暂存光标处的文件。
- `s` 暂存光标处的文件。
- `u` 取消暂存光标处的文件。
- `>` / `<` 内联显示或隐藏光标处文件的差异变化。

<p align="center">
  <img alt="Finding files in FZF" width="900" height="auto" src="images/fugitive-git.png">
</p>

查阅 `:h fugitive-staging-maps` 可获得更多信息。

## Git Blame

在当前文件运行 `:Git blame` 命令，vim-fugitive 可以显示一个拆分的问责窗口。这有助于追踪那些 BUG 是谁写的，接着就可以冲他/她怒吼（开个玩笑）。

在  `"git blame"` 模式下您可以做：

- `q` 关闭问责窗口。
- `A` 调整作者列大小。
- `C` 调整提交列大小。
- `D` 调整日期/时间列大小。

查阅 `:h :Git_blame` 可获得更多信息。

<p align="center">
  <img alt="Finding files in FZF" width="900" height="auto" src="images/fugitive-git-blame.png">
</p>

## Gdiffsplit

当您运行 `:Gdiffsplit` 命令后，vim-fugitive 会执行 `vimdiff`，比对索引或工作树中的版本与当前文件最新更改的区别。如果运行 `:Gdiffsplit <commit>`，vim-fugitive 则会根据 `<commit>` 中的版本来执行 `vimdiff`。

<p align="center">
  <img alt="Finding files in FZF" width="900" height="auto" src="images/fugitive-gdiffsplit.png">
</p>

由于您处于 `vimdiff` 模式中，因此您可以使用 `:diffput` 和 `:diffget` 来 *获取* 或 *输出* 差异。

## Gwrite 和 Gread

当您在更改文件后运行 `:Gwrite` 命令，vim-fugitive 将暂存更改，就像运行 `git add <current-file>` 一样。

当您在更改文件后运行 `:Gread` 命令，vim-fugitive 会将文件还原至更改前的状态，就像运行 `git checkout <current-file>` 一样。使用 `:Gread` 还有一个好处是操作可撤销。如果在运行 `:Gread` 后您改变主意，想要保留原来的更改，您只需要撤消（`u`），Vim 将撤回 `:Gread` 操作。要换作是在命令行中运行 `git checkout <current-file>`，就完成不了这种操作了。

## Gclog

当您运行 `:Gclog` 命令时，vim-fugitive 将显示提交历史记录，就像运行 `git log` 命令一样。Vim-fugitive 使用 Vim 的 quickfix 来完成此任务，因此您可以使用 `:cnext` 和 `:cprevious` 来遍历下一个或上一个日志信息。您还可以使用 `:copen` 和 `:cclose` 打开或关闭日志列表。

<p align="center">
  <img alt="Finding files in FZF" width="900" height="auto" src="images/fugitive-git-log.png">
</p>

在 `"git log"` 模式中，您可以做两件事：
- 查看树。
- 访问父级（上一个提交）。

您可以像 `git log` 命令一样，传递参数给 `:Gclog` 命令。如果您项目的提交历史记录很长，只想看最后三个提交，则可以运行 `:Gclog -3`。如果需要根据提交日期来筛选记录，可以运行类似 `:Gclog --after="January 1" --before="March 14"` 的命令。

## Vim-Fugitive 的更多功能

以上只是寥寥几个 vim-fugitive 功能的例子，您可以查阅 `:h fugitive.txt` 来了解更多有关 vim-fugitive 的信息。大多数流行的 git 命令可能都有 vim-fugitive 的优化版本，您只需在文档中查找它们。

如果您处于 vim-fugitive 的“特殊模式”（如 `:Git` 或 `:Git blame` 模式）中，按下 `g?` 可以了解当前有哪些可用的快捷键，Vim-fugitive 将为您所处的模式显示相应的 `:help` 窗口。棒极了！

## 聪明地学习 Vim 和 Git

每个人都有不同的 git 工作流，可能 vim-fugitive 非常合适您的工作流（也可能不适合）。总之，我强烈建议您试试上面列出的所有插件。可能还有一些其他的我没有列出来，都可以去试一试。

要让Vim-git的集成工作得更好，一个显而易见的办法就是去深入了解git。Git 本身是一个很庞大的主题，我只向您展示了它其中很小的一部分。好了，接下来谈谈如何使用 Vim 编译您的代码。

## 链接
- [目录](./directory.md)
- 上一部分 [Ch 17 - 折叠](./ch17_fold.md)
- 下一部分 [Ch 19 - 编译](./ch19_compile.md)