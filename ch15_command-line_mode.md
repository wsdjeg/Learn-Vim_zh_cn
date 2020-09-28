# 命令行模式

在前面的三个章节，你学习了如何使用搜索命令（`/`、`?`），替换命令（`:s`），全局搜索命令（`:g`），
以及拓展命令（`!`）。这些都是命令行模式下的示例。

在本章节，你将学习命令行模式下更多的技巧。

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

# Repeating the Previous Command

You can repeat the previous command-line command or external command with `@:`.

If you just ran `:s/foo/bar/g`, running `@:` repeats that substitution.

If you just ran `:.!tr '[a-z]' '[A-Z]'`, running `@:` repeats the last external command translation filter.

# Command-line Mode Shortcuts

While in the command-line mode, you can move to the left or to the right, one character at a time, with the `Left` or `Right` arrow.

If you need to move word-wise, use `Shift-Left` or `Shift-Right` (in some OS, you might have to use `Ctrl` instead of `Shift`).

To go to the start of the line, use `Ctrl-B`. To go to the end of the line, use `Ctrl-E`.

Similar to the insert mode, inside the command-line mode, you have three ways to delete characters:

```
Ctrl-H    Delete one character
Ctrl-W    Delete one word
Ctrl-U    Delete the entire line
```

# Register and Autocomplete

When programming, whenever possible, do not repeat if you can autocomplete it. This mindset will not only save you time but reduces the chances of typing the wrong characters.

You can insert texts from Vim register with `Ctrl-R` (the same way as the insert mode). If you have the string "foo" saved in the register "a" (`"a`), you can insert it by running `Ctrl-R a`. Everything that you can get from the register in the insert mode, you can do the same from the command-line mode.

You can also autocomplete commands. To autocomplete the `echo` command, while in the command-line mode, type "ec", then press `<Tab>`. You should see on the bottom left Vim commands starting with "ec" (example: `echo echoerr echohl echomsg econ`). To go to the next option, press either `<Tab>` or `Ctrl-N`. To go the previous option, press either `<Shift-Tab>` or `Ctrl-P`.

Some command-line commands accept file names as arguments. One example is `edit`. After typing the command, `:e` (don't forget the space), press `<Tab>`. Vim will list all the relevant file names.

# History Window

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
