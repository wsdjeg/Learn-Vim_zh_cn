# 第19章 编译

编译是许多编程语言的重要主题。在本章中，您将学习如何在 Vim 中编译。此外，您将看到如何利用好 Vim 的 `:make` 命令。

## 从命令行编译

您可以使用叹号运算符（`!`）进行编译。如果您需要使用 `g++` 来编译 `.cpp` 文件，可以运行：

```
:!g++ hello.cpp -o hello
```

但要每次手动指定文件名和输出文件名会非常繁琐和容易出错。而 makefile 是条可行之路。

## Make命令

Vim 有运行 makefile 的 `:make` 命令。当您运行它时，Vim 会在当前目录查找 makefile 并执行它。

在当前目录创建一个文件名为 `makefile` ，然后添加下列内容：

```
all:
	echo "Hello all"
foo:
	echo "Hello foo"
list_pls:
	ls
```

在 Vim 中运行：

```
:make
```

Vim 执行它的方式与从终端运行它的方式相同。`:make` 命令也接受终端中 `make` 命令的参数。运行：

```
:make foo
" Outputs "Hello foo"

:make list_pls
" Outputs the ls command result
```

如果命令执行异常，`:make` 命令将使用 Vim 的 `quickfix` 来存储这些错误。现在试着运行一个不存在的目标：

```
:make dontexist
```

您应该会看到该命令执行错误。运行 `quickfix` 命令 `:copen` 可以打开 `quickfix` 窗口来查看该错误：

```
|| make: *** No rule to make target `dontexist'.  Stop.
```

## 使用 Make 编译

让我们使用 makefile 来编译一个基本的 `.cpp` 程序。首先创建一个 `hello.cpp` 文件：

```
#include <iostream>

int main() {
    std::cout << "Hello!\n";
    return 0;
}
```

然后，更新 `makefile` 来编译和运行 `.cpp` 文件：

```
all:
	echo "build, run"
build:
	g++ hello.cpp -o hello
run:
	./hello
```

现在运行：

```
:make build
```

`g++` 将编译 `./hello.cpp` 并且生成 `./hello`。接着运行：

```
:make run
```

您应该会看到终端上打印出了 `"Hello!"`。

## 不同的Make程序

当您运行 `:make` 时，Vim 实际上会执行 `makeprg` 选项所设置的任何命令，您可以运行 `:set makeprg?` 来查看它：

```
makeprg=make
```

`:make` 的默认命令是外部的 `make` 命令。若想修改 `:make` 命令，使每次运行它时执行 `g++ <your-file-name>`，请运行：

```
:set makeprg=g++\ %
```

`\` 用于转义 `g++` 后的空格。Vim 中 `%` 符号代表当前文件。因此，`g++\ %` 命令等于运行 `g++ hello.cpp`。

转到 `./hello.cpp` 然后运行 `:make`，Vim 将编译 `hello.cpp` 并输出 `a.out`（因为您没有指定输出）。让我们重构一下，使用去掉扩展名的原始文件名来命名编译后的输出。运行下面的命令（或将它们添加到vimrc）：

```
:set makeprg=g++\ %\ -o\ %<
```

上面的命令分解如下：
- `g++\\ %` 如上所述，等同于运行 `g++ <your-file>`。
- `-o` 输出选项。
- `%<` 在 Vim 中代表了没有扩展名的当前文件名（如 `hello.cpp` 变成 `hello`）。

当您在 `./hello.cpp` 中运行 `:make` 时，它将编译为 `./hello`。要在 `./hello.cpp` 中快速地执行 `./hello`，可以运行 `:!./%<`。同样，它等同于运行 `:!./<无后缀的当前文件名>`。

查阅 `:h :compiler` 和 `:h write-compiler-plugin` 可以了解更多信息。

## 保存时自动编译

有了自动化编译，您可以让生活更加轻松。回想一下，您可以使用 Vim 的 `autocmd` 来根据某些事件自动执行操作。例如，要自动在每次保存后编译 `.cpp` 文件，您可以将下面内容添加到vimrc：

```
:autocmd BufWritePost *.cpp make
```

现在您每次保存 `.cpp` 文件后，Vim 都将自动执行 `make` 命令。

## 切换编译器

Vim 有一个 `:compiler` 命令可以快速切换编译器。您的 Vim 可能附带了一些预构建的编译配置。要检查您拥有哪些编译器，请运行：

```
:e $VIMRUNTIME/compilers/<tab>
```

您应该会看到一个不同编程语言的编译器列表。

若要使用 `:compiler` 命令，假设您有一个 ruby 文件 `hello.rb`，内容是：

```
puts "Hello ruby"
```

回想一下，如果运行 `:make`，Vim 将执行赋值给 `makeprg` 的任何命令（默认是 `make`）。如果您运行：

```
:compiler ruby
```

Vim 执行 `$VIMRUNTIME/compiler/ruby.vim` 脚本，并将 `makeprg` 更改为使用 `ruby` 命令。现在如果您运行 `:set makeprg?`，它会显示 `makeprg=ruby`（这取决于您 `$VIMRUNTIME/compiler/ruby.vim` 里的内容，如果您有其他自定义的 ruby 编译器，您的结果可能会有不同）。`:compiler <your-lang>` 命令允许您快速切换至其他编译器。如果您的项目使用多种语言，这会非常有用。

您不必使用 `:compiler` 或 `makeprg` 来编译程序。您可以运行测试脚本、分析文件、发送信号或任何您想要的内容。

## 创建自定义编译器

让我们来创建一个简单的 Typescript 编译器。先在您的设备上安装 Typescript（`npm install -g typescript`），安装完后您将有 `tsc` 命令。如果您之前没有尝试过 typescript，`tsc` 将 Typescript 文件编译成 Javascript 文件。假设您有一个 `hello.ts` 文件：

```
const hello = "hello";
console.log(hello);
```

运行 `tsc hello.ts` 后，它将被编译成 `hello.js`。然而，如果您的 `hello.ts` 文件中有如下内容：

```
const hello = "hello";
hello = "hello again";
console.log(hello);
```

这会抛出错误，因为不能更改一个 `const` 变量。运行 `tsc hello.ts` 的错误如下：

```
hello.ts:2:1 - error TS2588: Cannot assign to 'person' because it is a constant.

2 person = "hello again";
  ~~~~~~


Found 1 error.
```

要创建一个简单的 Typescript 编译器，请在您的 `~/.vim/` 目录中新添加一个 `compiler` 目录（即 `~/.vim/compiler/`），接着创建 `typescript.vim` 文件（即 `~/.vim/compiler/typescript.vim`），并添加如下内容：

```
CompilerSet makeprg=tsc
CompilerSet errorformat=%f:\ %m
```

第一行将 `makeprg` 设置为运行 `tsc` 命令。第二行将错误格式设置为显示文件（`%f`），后跟冒号（`:`）和转义的空格（`\ `），最后是错误消息（`%m`）。查阅 `:h errorformat` 可了解更多关于错误格式的信息。

您还可以阅读一些预制的编译器，看看它们是如何实现的。输入 `:e $VIMRUNTIME/compiler/<some-language>.vim` 查看。

有些插件可能会干扰 Typescript 文件，可以使用 `--noplugin` 标志以零插件的形式打开`hello.ts` 文件：

```
vim --noplugin hello.ts
```

检查 `makeprg`：

```
:set makeprg?
```

它应该会显示默认的 `make` 程序。要使用新的 Typescript 编译器，请运行：

```
:compiler typescript
```

当您运行 `:set makeprg?` 时，它应该会显示 `tsc` 了。我们来测试一下：

```
:make %
```

回想一下，`%` 代表当前文件。看看您的 Typescript 编译器是否如预期一样工作。运行 `:copen` 可以查看错误列表。

## 异步编译器

有时编译可能需要很长时间。在等待编译时，您不会想眼睁睁盯着已冻结的 Vim 的。如果可以异步编译，就可以在编译期间继续使用 Vim 了，岂不美哉？

幸运的是，有插件来运行异步进程。有两个比较好的是：

- [vim-dispatch](https://github.com/tpope/vim-dispatch)
- [asyncrun.vim](https://github.com/skywind3000/asyncrun.vim)

在这一章中，我将介绍 vim-dispatch，但我强烈建议您尝试上述列表中所有插件。

*Vim 和 NeoVim 实际上都支持异步作业，但它们超出了本章的范围。如果您好奇，可以查阅 `:h job-channel-overview.txt`。*

## 插件：Vim-dispatch

Vim-dispatch 有几个命令，最主要的两个是 `:Make` 和 `:Dispatch`。

### 异步Make

Vim-dispatch 的 `:Make` 命令与 Vim 的 `:make` 相似，但它以异步方式运行。如果您正处于 Javascript 项目中，并且需要运行 `npm t`，可以将 `makeprg` 设置为：

```
:set makeprg=npm\\ t
```

如果运行：

```
:make
```

Vim 将执行 `npm t`。但同时，您只能盯着冻结了的屏幕。有了 vim-dispatch，您只需要运行：

```
:Make
```

Vim 将启用后台进程异步运行 `npm t`，同时您还能在 Vim 中继续编辑您的文本。棒极了！

### 异步调度（Dispatch）

`:Dispatch` 命令的工作方式和 `:compiler` 及 `:!` 类似，它可以在Vim中运行任意外部命令。

假设您在 ruby spec 文件中，需要执行测试，可以运行：

```
:Dispatch rspec %
```

Vim 将对当前文件异步运行 `rspec` 命令。

### 自动调度

Vim-dispatch 有一个缓冲区变量`b:dispatch`，您可以配置它来自动执行特定命令，您可以利用 `autocmd`和它一起工作。如果在您的 vimrc 中添加如下内容：

```
autocmd BufEnter *_spec.rb let b:dispatch = 'bundle exec rspec %'
```

现在每当您进入（`BufEnter`）一个以 `_spec.rb` 结尾的文件，运行`:Dispatch` 将自动执行 `bundle exec rspec <your-current-ruby-spec-file>`。

## 聪明地学习编译

在本章中，您了解到可以使用 `make` 和 `compiler` 命令从Vim内部异步运行 *任何* 进程，以完善您的编程工作流程。Vim 拥有通过其他程序来扩展自身的能力，这使其变得强大。

## 链接
- [目录](./directory.md)
- 上一部分 [Ch 18 - Git](./ch18_git.md)
- 下一部分 [Ch 20 - 视图、会话和 Viminfo](./ch20_views_sessions_viminfo.md)