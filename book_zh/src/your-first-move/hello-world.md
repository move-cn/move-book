# 你好，世界！

在本章中，您将学习如何创建一个新的包，编写一个简单的模块，进行编译，并使用Move CLI运行测试。请确保您已经[安装了Sui](./../before-we-begin/install-sui.md)并设置了[IDE环境](./../before-we-begin/ide-support.md)。运行以下命令来测试Sui是否正确安装。

```bash
# 它应该打印出客户端版本号。例如：sui-client 1.22.0-036299745。
sui client --version
```

> Move CLI是Move语言的命令行界面；它内置于Sui二进制文件中，提供了一组命令来管理包、编译和测试代码。

本章的结构如下：

- [创建一个新的包](#创建一个新的包)
- [目录结构](#目录结构)
- [编译包](#编译包)
- [运行测试](#运行测试)

## 创建一个新的包

要创建一个新的程序，我们将使用`sui move new`命令，后面跟上应用程序的名称。我们的第一个程序将被命名为`hello_world`。

> 注意：在本章和其他章节中，如果您看到以`$`（美元符号）开头的代码块，表示应该在终端中运行以下命令。不要包含这个符号。这是一种在终端环境中显示命令的常见方式。

```bash
$ sui move new hello_world
```

`sui move`命令可以访问Move CLI，它是一个内置的编译器、测试运行器和用于处理Move的实用工具。`new`命令后面跟上包的名称将在新的文件夹中创建一个新的包。在我们的例子中，文件夹的名称是"hello_world"。

我们可以查看文件夹的内容，以确认包已成功创建。

```bash
$ ls -l hello_world
Move.toml
sources
tests
```

## 目录结构

Move CLI将创建应用程序的基本结构，并预先创建目录结构和所有必要的文件。让我们看看里面的内容。

```plaintext
hello_world
├── Move.toml
├── sources
│   └── hello_world.move
└── tests
    └── hello_world_tests.move
```

### Manifest文件

`Move.toml`文件被称为[包清单](./../concepts/manifest.md)，它包含包的定义和配置设置。它被Move编译器用来管理包的元数据、获取依赖项和注册命名地址。我们将在[概念](./../concepts)章节中详细解释它。

> 默认情况下，包具有一个命名地址 - 包的名称。

```toml
[addresses]
hello_world = "0x0"
```

### 源代码

`sources/`目录包含源文件。Move源文件的扩展名是`.move`，通常以文件中定义的模块命名。例如，在我们的例子中，文件名是`hello_world.move`，Move CLI已经在其中放置了注释的代码：

```move
/*
/// 模块：hello_world
module hello_world::hello_world {

}
*/
```

注释掉的代码是一个模块定义，它以关键字`module`开头，后面是命名地址（或地址字面量），然后是模块名称。模块名称是模块的唯一标识符，并且在包内必须是唯一的。模块名称用于从其他模块或交易中引用该模块。

<!-- 模块名称必须是有效的Move标识符：字母数字字符和下划线用于分隔单词。常见的约定是以snake_case（全部小写，使用下划线分隔）命名模块（和函数）。编码规范对于代码的可读性和可维护性### 源代码

`sources/`目录包含源文件。Move源文件的扩展名是`.move`，通常以文件中定义的模块命名。在我们的例子中，文件名是`hello_world.move`，Move CLI已经在其中放置了注释的代码：

```move
/*
/// 模块：hello_world
module hello_world::hello_world {

}
*/
```

注释掉的代码是一个模块定义，它以关键字`module`开头，后面是命名地址（或地址字面量），然后是模块名称。模块名称是模块的唯一标识符，并且在包内必须是唯一的。模块名称用于从其他模块或交易中引用该模块。

> 模块名称必须是有效的Move标识符：字母数字字符和下划线用于分隔单词。常见的约定是使用snake_case（全部小写，使用下划线分隔）命名模块（和函数）。编码规范对于代码的可读性和可维护性至关重要。

### 测试代码

`tests/`目录包含测试文件。测试文件使用Move代码编写，用于测试模块和函数的行为。在我们的例子中，文件名是`hello_world_tests.move`，其中包含了一些示例测试代码。

```move
/*
/// 模块：hello_world_tests
module hello_world::hello_world_tests {

}
*/
```

与源文件类似，测试文件也是一个模块定义，以关键字`module`开头，后面是命名地址和模块名称。

> 在Move中，测试是以模块的形式组织的，这样可以更好地与要测试的模块关联起来，并使测试代码更易于阅读和维护。

## 编译包

现在我们已经创建了一个新的包，并且了解了包的基本结构，让我们尝试编译它。

我们将使用`sui move compile`命令来编译包。在`hello_world`目录中运行以下命令：

```bash
$ sui move compile
```

Move CLI将读取`Move.toml`文件并编译`sources/`目录中的所有Move源文件。如果编译成功，它将在`target/`目录中生成`.mv`文件。

```plaintext
hello_world
├── Move.toml
├── sources
│   └── hello_world.move
├── target
│   └── deps
│       └── hello_world.mv
└── tests
    └── hello_world_tests.move
```

在我们的例子中，编译器生成了一个名为`hello_world.mv`的文件，并将其放在`target/deps/`目录中。

> 编译器还会验证Move源代码中的语法和类型错误，并在出现错误时生成适当的错误消息。

## 运行测试

现在我们已经编译了包，让我们尝试运行测试。

我们将使用`sui move test`命令来运行包中的所有测试。在`hello_world`目录中运行以下命令：

```bash
$ sui move test
```

Move CLI将读取`Move.toml`文件并运行`tests/`目录中的所有测试。它将编译测试文件并执行其中的测试函数。如果所有测试通过，它将输出一个成功的消息。

```plaintext
Running 1 test from 1 module
test hello_world::hello_world_tests::test_hello_world ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

在我们的例子中，我们只有一个测试函数`test_hello_world`，它成功通过了。

> Move CLI提供了许多选项和标志，以便您可以更精细地控制测试的行为。您可以使用`sui move test --help`命令获取更多信息。

恭喜！您已成功创建一个新的包，编译了源代码，并运行了