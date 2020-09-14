# Macros

在编辑文件的时候，你会发现有时候你反复在做一些相同的动作。如果你仅做一次并在你需要的时候调用这些动作岂不是会更好。通过 Vim 的宏命令，你可以将一些动作录下到 Vim 寄存器。

在本章中，你将会学习到如何通过宏命令自动完成一些普通的任务（另外，看你的文件在自己编辑是一件很酷的事情）。

# Basic Macros

宏命令的基本语法如下：

```
qa                     Start recording a macro in register a
q (while recording)    Stop recording macro
```

你可以使用小写字母 （a-z）去存储宏命令。并通过如下的命令去调用：

```
@a    Execute macro from register a
@@    Execute the last executed macros
```

假设你有如下的文本，你打算将每一行中的字母都变为大写。

```
hello
vim
macros
are
awesome
```

将你的光标移动到 “hello”栏的行首，并执行：
```
qa0gU$jq
```

上面命令的分解如下：

- `qa` 开始记录一个宏定义并存储在 a 寄存器。
- `0` 移动到行首。
- `gU$` 将光标到行尾的字母变为大写。
- `j` 移动到下一行。
- `q` 停止记录。

调用 `@a` 去执行该宏命令。就像其他的宏命令，你也可以为该命令加一个计数。例如，你可以执行 `3@a` 去执行 `a` 命令3次。你可以执行 `3@@` 去执行最后一次执行过的宏命令3次。

# Safety Guard

在执行遇到错误的时候，宏命令会自动停止。假如你有如下文本：

```
a. chocolate donut
b. mochi donut
c. powdered sugar donut
d. plain donut
```

假如你想将每一行的第一个词变为大写，你可以使用如下的宏命令：
```
qa0W~jq
```

上面命令的分解如下：
- `qa` 开始记录一个宏定义并存储在 a 寄存器。
- `0` 移动到行首。
- `W` 移动到下一个单词。
- `~` 将光标选中的单词变为大写。
- `j` 移动到下一行。
- `q` 停止记录。

我喜欢对宏命令进行很多次的调用，所以通常我使用 `99@a` 命令去执行该宏命令99次。当 Vim 在最后一行执行 `j` 命令的时候，它会发现以及没有下一行可以继续，遇到执行的错误，因此宏命令会停止。

实际上，遇到错误自动停止运行是一个很好的特性。否则，Vim 会继续执行该命令99次，尽管它以及执行到最后一行了。

# Command Line Macro

在正常模式执行 `@a` 并不是调用宏命令调用的唯一方式。你也可以在命令行执行 `：normal @a` 。`：normal` 会将任何用户添加的参数作为命令去执行。添加 `@a`，和在 normal mode 执行 `@a` 的效果是一样的。

`:normal` 命令也支持范围参数。你可以在选择的范围内去执行宏命令。如果你只想在第二行和第三行执行宏命令，你可以执行 `：2,3 normal @a`。我会在后续的章节中介绍在命令行中执行的命令。

# Executing a Macro Across Multiple Files

假如你有很多的 `.txt` 文件，每一个文件包含不同的内容。而且，你只想将包含有 “donut” 单词的行的第一个单词变为大写。那么，该如何在很多文件中特定的行执行执行变大写的操作呢？

第一个文件:
```
# savory.txt
a. cheddar jalapeno donut
b. mac n cheese donut
c. fried dumpling
```

第二个文件:
```
# sweet.txt
a. chocolate donut
b. chocolate pancake
c. powdered sugar donut
```

第三个文件:
```
# plain.txt
a. wheat bread
b. plain donut
```

你可以这么做:
- `:args *.txt` 查找当前目录下的所有 `.txt` 文件。
- `:argdo g/donut/normal @a` 在所有 `:args` 中包含的文件里执行一个全局命令 `g/donut/normal @a`。
- `:argdo update` 在所有 `:args` 中包含的文件里执行 `update` 命令会将修改后的内容保存下来。

如果你对全局命令 `:g/donut/normal @a` 不是很了解的话，该命令会在包含有 `/donut/` 中的所有行执行`normal @a` 命令。我会在后面的章节中介绍全局命令。

# Recursive Macro
你可以递归地执行宏命令，通过在记录宏命令时调用相同的宏来实现。假如你有如下文本，你希望改变第一个单词的大小写：

```
a. chocolate donut
b. mochi donut
c. powdered sugar donut
d. plain donut

```
如下命令会递归地执行:
```
qaqqa0W~j@aq
```

上面命令的分解如下：

- `qaq` 记录一个空白的宏命令到 “a” 。把宏命令记录在一个空白的命令中是必须的，因为你不会想将该命令包含有任何其他的东西。
- `qa` 开始录入宏命令到寄存器 “a”。
- `0` 移动到行首。
- `W` 移动到下一个单词。
- `~` 改变光标选中的单词的大小写。
- `j` 移动到下一行。
- `@a` 执行宏命令 “a”。当你记录该宏命令时，`@a` 应该是空白的，因为你刚刚调用了 `qaq`。
- `q` 停止记录。

现在，让我们来调用 `@a` 来查看 Vim 如何递归的调用该宏命令。

宏命令是如何知道何时停止呢？当宏执行到最后一行时，它会尝试 `j` 命令，发现已经没有下一行了，就会停止执行。

# Appending a Macro

如果你想在一个已经录制好的宏定义中添加更多的操作，与其重新录入它，你可以选择修改它。在寄存器一章中，你学习了你可以使用一个已知寄存器的大写字母来添加一个新的寄存器。为了在寄存器“a”中添加更多的操作，你可以使用“A”。假设你不仅希望将第一个单词变为大写，你也希望在每一行末尾添加一个句点。

假设你在寄存器“a”中有如下的命令：
```
0W~
```
你可以这样做:
```
qAA.<esc>q
```
分解如下:
- `qA` 开始在寄存器 “A” 中记录宏命令。
- `A.<esc>` 在行的末尾（`A`）假如一个句点，并且退出插入模式。
- `q` 停止记录宏命令。

现在，当你执行 `@a` 时，它会跳到行的第一个字符（`0`），跳到下一个单词（`W`），改变光标选中的字母的大小写（`~`），移动到最后一行并且转到插入模式（`A`），写入一个句点（`.`），退出插入模式（`<esc>`）。

# Amending a Macro

在已存在的宏定义的末尾添加新的动作是一个很好的功能，但假如你希望在一个宏命令的中间添加动作该怎么做呢？本节，我会向你展示如何修改一个宏。

假设，在改变第一个单词的大小写和在末尾加入一个句点之间，你想要在单词 “donut” 之前加入 “deep fried”（因为唯一比甜甜圈好的东西就是炸甜甜圈）。

我会重新使用之前一节使用过的文本:
```
a. chocolate donut
b. mochi donut
c. powdered sugar donut
d. plain donut
```

首先，让我们通过 `:put a` 调用一个已经录制好的宏命令（假设你已经有了上一节中使用过的宏命令）：

```
0W~A.^[
```

`^[` 是什么意思呢？不记得了吗，你之前执行过 `0W~A.<esc>`。 `^[` 是 Vim 的内部指令，表示 `<esc>`。通过这些指定的键值组合，Vim 知道这些是内部代码的一些替代。一些常见的内部指令具有类似的替代，例如 `<esc>`，`<backspace>`，`<enter>`。还有一些其他的键值组合，但这不是本章的内容。
  
回到宏命令，在改变大小写之后的键后面（`~`），让我们添加（`$`）来移动光标到行末，回退一个单词（`b`），进入插入模式（`i`），输入“deep fried ”（别忽略“fried ” 后面的这个空格），之后退出插入模式（`<esc>`）。

完整的命令如下:
```
0W~$bideep fried <esc>A.^[
```

这里有一个问题，Vim 不能理解 `<esc>`。所以你需要将其替换为内部代码的形式。在插入模式，在按下`<esc>`后按下 `Ctrl-v`，Vim 会打印 `^[`。 `Ctrl-v` 是一个插入模式的操作符，可以逐字地插入一个非数字字符。你的宏命令应该如下:

```
0W~$bideep fried ^[A.^[
```


To add the amended instruction into register "a", you can do it the same way as adding a new entry into a named register. At the start of the line, run `"ay$`. This tells Vim that you're using the named register "a" (`"a`) to store the yanked text from the current position to the end of the line (`y$`).

Now when you execute `@a`, your macro will toggle the case of the first word, add "deep fried " before "donut", and add a "." at the end of the line.

An alternative way to amend a macro is to use a command line expression. Do `:let @a="`, then do `Ctrl-r Ctrl-r a`, this will literally paste the content of register "a". Finally, don't forget to close the double quotes (`"`). If you need to insert special characters using internal codes while editing a command line expression, you can use `Ctrl-v`.

# Macro Redundancy

You can easily duplicate macros from one register to another. For example, to duplicate a macro in register "a" to register "z", you can do `:let @z = @a`. `@a` represents the content of register "a". Now if you run `@z`, it does the exact same actions as `@a`.

I find creating a redundancy useful on my most frequently used macros. In my workflow, I usually record macros in the first seven alphabetical letters (a-g) and I often replace them without much thought. If I move the useful macros towards the end of the alphabets, I can preserve them without worrying that I might accidentally replace them.

# Series vs Parallel Macro

Vim can execute macros in series and parallel. Suppose you have this text:

```
import { FUNC1 } from "library1";
import { FUNC2 } from "library2";
import { FUNC3 } from "library3";
import { FUNC4 } from "library4";
import { FUNC5 } from "library5";
```

If you want to record a macro to lowercase all the uppercased "FUNC", this macro should work:

```
qa0f{gui{jq
```

Here is the breakdown:
- `qa` starts recording in register "a".
- `0` goes to first line.
- `f{` finds the first instance of "{".
- `gui{` lowercases (`gu`) the text inside the bracket text-object (`i{`).
- `j` goes down one line.
- `q` stops macro recording.

Now you can run `99@a` to execute it on the remaining lines. However, what if you have this import expression inside your file?

```
import { FUNC1 } from "library1";
import { FUNC2 } from "library2";
import { FUNC3 } from "library3";
import foo from "bar";
import { FUNC4 } from "library4";
import { FUNC5 } from "library5";
```

Running `99@a`, only executes the macro three times. It does not execute the macro on last two lines because the execution fails to run `f{` on the "foo" line. This is expected when running the macro in series. You can always go to the next line where "FUNC4" is and replay that macro again. But what if you want to get everything done in one go? You can run the macro in parallel.

Recall from earlier section that macros can be executed using the  command line command `:normal` (ex: `:3,5 normal @a` to execute macro "a" on lines 3-5). If you run `:1,$ normal @a`, you will see that the macro is being executed on all lines except the "foo" line. It works!

Although internally Vim does not actually run the macros in parallel, outwardly, it behaves like such. Vim executes `@a` *independently* on each line from the first line to the last line (`1,$`). Since Vim executes these macros independently, each line does not know that one of the macro executions had failed on the "foo" line.

# Learn Macros the Smart Way

Many things you do in editing are repetitive. To get better at editing, get into the habit of detecting repetitive actions. Use macros (or dot command) so you don't have to perform the same action twice. Almost everything that you can do in Vim can be done with macros.

In the beginning, I find it very awkward to write macros, but don't give up. With enough practice, you will get into the habit of automating everything.

You might find it helpful to use mnemonics to help remember your macros. If you have a macro that creates a function, use the "f" register (`qf`). If you have a macro for numerical operations, the "n" register may be a good fit (`qn`). Name it with the *first named register* that comes to your mind when you think of that operation. I also find that "q" register makes a good default macro register because `qq` does not require much brain power to use. Lastly, I like to increment my macros in alphabetical orders, like `qa`, then `qb`, then `qc`, and so on. Find a method that works best for you.
