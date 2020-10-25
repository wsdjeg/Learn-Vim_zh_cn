# 命令行模式

在前面的三个章节，你学习了如何使用搜索命令（`/`、`?`），替换命令（`:s`），全局搜索命令（`:g`），
以及拓展命令（`!`）。这些都是命令行模式下的示例。

在本章节，你将学习命令行模式下更多的技巧。

<!-- vim-markdown-toc GFM -->

  - [进入、离开命令行模式](#进入离开命令行模式)
  - [重复上一次命令](#重复上一次命令)
  - [命令行模式快捷键](#命令行模式快捷键)
  - [寄存器与自动完成](#寄存器与自动完成)
  - [历史窗口](#历史窗口)
- [Command-line Window](#command-line-window)
- [Learn Command-line Mode the Smart Way](#learn-command-line-mode-the-smart-way)

<!-- vim-markdown-toc -->

## 进入、离开命令行模式

命令行模式同输入模式、可视模式类似，也是一种 Vim 模式，在这种模式下，光标会到屏幕最下方，
此时可以输入一些命令。

有四种进入命令行模式的方式：

- 搜索（`/`、`?`）
- 执行Vim命令（`:`）
- 拓展命令（`!`）

可以从普通模式、可视模式进入命令行模式。

如果需要退出命令行模式，可以使用 `<esc>`、`Ctrl-C` 或者 `Ctrl-[` 按键。

_Sometimes other literatures might refer the "Command-line command" as "Ex command" and the "External command" as "filter command" or "bang operator"._

## 重复上一次命令

你可以通过快捷键 `@:` 重复执行上一次命令。

如果之前执行的命令是 `:s/foo/bar/g`，按下 `@:` 将重复这一替换。

如果之前执行的命令是 `:.!tr '[a-z]' '[A-Z]'`，按下 `@:` 将重复这一拓展命令。

## 命令行模式快捷键

当处在命令行模式下，你可以使用左右方向键移动光标，一次一个字符。

如果需要一次移动一个词的位置，使用 `Shift-Left` 和 `Shift-Right` (在某些系统下，你可能需要使用 `ctrl` 键，而非 `Shift` 键).

使用 `Ctrl-b` 快捷键移动光标至行首，`Ctrl-e` 快捷键移动光标至行尾。

类似于插入模式，在命令行模式下，有三个快捷键可以删除字符：

```
Ctrl-H    Delete one character
Ctrl-W    Delete one word
Ctrl-U    Delete the entire line
```

## 寄存器与自动完成

在编程过程中，无论何时，如果可以自动完成，就不需要重复操作。这一原则将不仅会为你节省很多时间，并且可以避免输入出错。

可以从 Vim 的寄存器中输入字符（同插入模式一样）。如果寄存器 "a"（`"a`）中的内容是 "foo"，那么就可以通过快捷键 `Ctrl-r a` 输入。
任何可以在输入模式下获取到的寄存器内容，在命令行模式下同样可以获取到。

你也可以自动补全命令。比如，如果需要补全 `echo` 命令，在命令行模式下，按下 `ec`，然后按下 `<Tab>` 键，就可以补全了。
你可以在状态栏上看到其他的以 `ec` 开头的命令，比如（`echo echoerr echohl echomsg econ`）。
可以使用 `<Tab>` 或者 `Ctrl-n` 选择下一个补全命令。
也可以使用 `<Shift-Tab>` 或者 `Ctrl-p` 选择上一个补全命令。

一些命令支持接受文件名作为其参数。
其中一个例子就是 `edit` 命令。
按下这一命令后，`:e`（不要忘记按下空格键），继续按下 `<Tab>` 键，Vim 将列出所有相对路径下的文件。

## 历史窗口

你可以使用窗口打开并查看历史执行的命令，和搜索内容（确保你的 Vim 支持 `+cmdline_hist` 特性，可以使用 `vim --version` 查看当前 Vim 支持的特性）。

可以通过命令 `:his :` 打开命令历史窗口：

You can view the histoy of command-line commands and search terms (make sure that your Vim build has `+cmdline_hist` when you run `vim --version`).

To open the command-line history, run `:his :`:

```
#  cmd history
2  e file1.txt
3  g/foo/d
4  s/foo/bar/g
```

Vim lists the history of all the `:` commands you run. By default, Vim stores the last 50 commands. To change the amount of the entries that Vim remembers to 100, you can run `:set history=100`.

When you are in the command-line mode, you can traverse this history list by pressing `Up` and `Down` button. Suppose you had the command-line command history that looks like:

```
51  s/foo/bar/g
52  s/foo/baz/g
53  s/foo//g
```

If you press `:` then press `Up` once, you'll see `:s/foo//g`. Press `Up` one more time to see `:s/foo/baz/g`. Vim goes up the history list.

Similarly, to view the search history, run `:his /`. You can also traverse the history stack by pressing `Up` or `Down` after running the history command `/`.

Vim is smart enough to distinguish different histories. If you press `Up` or `Down` after pressing `:`, Vim automatically the command history. If you press `Up` or `Down` after pressing `/`, Vim automatically searches the search history.

# Command-line Window

The history window displays the list of previously used command-line commands, but you can't execute the command from the history window. To execute a command while browsing, use the _command-line window_. There are three different command-line windows:

```
q:    Command-line window
q/    Forward search window
q?    Backward search window
```

Run `q:` to launch the command-line window for command-line commands. Vim will launch a new window at the bottom of the screen. You can traverse upward with the `Up` or `Ctrl-P` keys and traverse downward with the `Down` or `Ctrl-N` keys. If you press `<Return>`, Vim executes that command. To quit the command-line window, either press `Ctrl-C`, `Ctrl-W C`, or type `:quit`.

Similarly, to launch the command-line window for search, run `q/` to search forward and `q?` to search backward.

# Learn Command-line Mode the Smart Way

Compared to the other three modes, the command-line mode is like the Swiss Army knife of text editing. You can edit text, modify files, and execute commands, just to name a few. This chapter is a collection of odds and ends of the command-line mode. It also brings Vim modes into closure. Now that you know how to use the normal, insert, visual, and command-line mode you can edit text with Vim faster than ever.

It's time to move away from Vim modes and learn how to do a faster navigation with Vim tags.
