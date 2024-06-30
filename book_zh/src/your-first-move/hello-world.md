# 你好，世界！

在本章中，您将学习如何创建一个新的包，编写一个简单的模块，进行编译，并使用Move CLI运行测试。请确保您已经[安装了Sui](./../before-we-begin/install-sui.md)并设置了[ IDE 环境](./../before-we-begin/ide-support.md)。运行以下命令来测试 Sui 是否正确安装。

```bash
# 它应该打印出客户端版本号。例如：sui-client 1.22.0-036299745。
sui client --version
```

> Move CLI 是 Move 语言的命令行界面；它内置于 Sui 二进制文件中，提供了一组命令来管理包、编译和测试代码。

本章的结构如下：

- [创建一个新的包](#创建一个新的包)
- [目录结构](#目录结构)
- [编译包](#编译包)
- [运行测试](#运行测试)

## 创建一个新的包

要创建一个新的程序，我们将使用 `sui move new` 命令，后面跟上应用程序的名称。我们的第一个程序将被命名为 `hello_world`。

> 注意：在本章和其他章节中，如果您看到以 `$`（美元符号）开头的代码块，表示应该在终端中运行以下命令。不要包含这个符号。这是一种在终端环境中显示命令的常见方式。

```bash
$ sui move new hello_world
```

`sui move`命令可以访问 Move CLI，它是一个内置的编译器、测试运行器和用于处理 Move 的实用工具。`new` 命令后面跟上包的名称将在新的文件夹中创建一个新的包。在我们的例子中，文件夹的名称是"hello_world"。

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

### Manifest

`Move.toml`文件被称为[包清单](./../concepts/manifest.md)，它包含包的定义和配置设置。它被Move编译器用来管理包的元数据、获取依赖项和注册命名地址。我们将在[概念](./../concepts)章节中详细解释它。

> 默认情况下，包具有一个以包名称命名的地址。

```toml
[addresses]
hello_world = "0x0"
```

### 源代码

`sources/` 目录包含源文件。Move源文件的扩展名是`.move`，通常以文件中定义的模块命名。例如，在我们的例子中，文件名是 `hello_world.move`，Move CLI 已经在其中放置了注释的代码：

```move
/*
/// 模块：hello_world
module hello_world::hello_world {

}
*/
```


> `/*` 和 `*/` 是 Move 中的注释符。它们之间的所有内容都会被编译器忽略，可用于文档或笔记。我们在 [基本语法](./../move-basics/comments.md) 中解释了所有注释代码的方式。


注释掉的代码是一个模块定义，它以关键字 `module` 开头，后面是命名地址（或地址字面量），然后是模块名称。模块名称是模块在包中的唯一标识符，并且在包内必须是唯一的。模块名称用于从其他模块或交易中引用该模块。


### 测试代码

`tests/` 目录包含包测试。编译器在常规构建过程中将排除这些文件，但在*测试*和*开发*模式下使用它们。这些测试用 Move 语言编写，并标有 `#[test]` 属性。测试可以分组在单独的模块中（通常命名为 *模块名*_tests.move），或者放在它们所测试的模块内部。

模块、导入、常量和函数可以用 `#[test_only]` 注解。这个属性用于在构建过程中排除模块、函数或导入。当你想为测试添加辅助功能，但不希望将它们包含在将发布到链上的代码中时将大有用处。

*hello_world_tests.move* 文件包含一个被注释掉的测试模块模板：

```move
/*
#[test_only]
module hello_world::hello_world_tests {
    // uncomment this line to import the module
    // use hello_world::hello_world;

    const ENotImplemented: u64 = 0;

    #[test]
    fun test_hello_world() {
        // pass
    }

    #[test, expected_failure(abort_code = hello_world::hello_world_tests::ENotImplemented)]
    fun test_hello_world_fail() {
        abort ENotImplemented
    }
}
*/
```

### 其他文件夹

此外，Move CLI 支持 `examples/` 文件夹。该文件夹中的文件处理方式与放置在 `tests/` 文件夹下的文件类似 - 它们只在*测试*和*开发*模式下被构建。这些文件旨在展示如何使用该包或如何将其与其他包集成。最常见的用例是用于文档目的和库包。


## 编译包

Move 是一种编译型语言，因此它需要将源文件编译成 Move 字节码。字节码只包含关于模块、其成员和类型的必要信息，同时排除了注释和某些标识符（例如，常量的标识符）。

为了演示这些特性，让我们用以下内容替换 *sources/hello_world.move* 文件的内容：

```move
/// 命名地址 `hello_world` 下的 `hello_world` 模块。
/// 命名地址在 `Move.toml` 中设置。
module hello_world::hello_world {
    // 从标准库导入 `String` 类型
    use std::string::String;
    /// 返回 "Hello, World!" 作为 `String`。
    public fun hello_world(): String {
        b"Hello, World!".to_string()
    }
}
```

在编译过程中，代码被构建但不会运行。编译后的包只包含可以被其他模块调用或在交易中使用的函数。我们将在 [概念 (concepts)](./../concepts) 章节中解释这些概念。现在，让我们看看运行 *sui move build* 时会发生什么。

```bash
# 在 `hello_world` 文件夹中运行
$ sui move build
# 或者，如果你处于该文件夹之外
$ sui move build --path hello_world
```

它应该在你的控制台输出以下消息：

```plaintext
UPDATING GIT DEPENDENCY https://github.com/MystenLabs/sui.git
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING hello_world
```

在编译过程中，Move 编译器会自动创建一个 build 文件夹，其中放置所有获取和编译的依赖项，以及当前包的模块的字节码。

> 如果你使用版本控制系统（如 Git），build 文件夹应该被忽略。例如，你应该使用 `.gitignore` 文件并在其中添加 `build`。

## 运行测试

在开始测试之前，我们应该添加一个测试。Move 编译器支持用 Move 编写的测试，并提供执行环境。测试可以放在源文件中，也可以放在 `tests/` 文件夹中。测试用 `#[test]` 属性标记，编译器会自动发现它们。我们将在 [测试](./../move-basics/testing.md) 部分深入解释测试。

请用以下内容替换 `tests/hello_world_tests.move`：

```move
#[test_only]
module hello_world::hello_world_tests {
    use hello_world::hello_world;
    #[test]
    fun test_hello_world() {
        assert!(hello_world::hello_world() == b"Hello, World!".to_string(), 0);
    }
}
```

这里我们导入 `hello_world` 模块，并调用其 `hello_world` 函数来测试输出是否为字符串 "Hello, World!"。现在我们已经准备好了测试，让我们在测试模式下编译并运行测试。Move CLI 有 `test` 命令用于此目的：

```bash
$ sui move test
```

将有以下内容输出：

```plaintext
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING hello_world
Running Move unit tests
[ PASS    ] 0x0::hello_world_tests::test_hello_world
Test result: OK. Total tests: 1; passed: 1; failed: 0
```

如果你在包文件夹外运行测试，可以指定包的路径：

```bash
$ sui move test --path hello_world
```

你还可以通过指定一个字符串来一次运行单个或多个测试。所有包含该字符串的测试都将被运行：

```bash
$ sui move test test_hello
```

## 下一步

在本节中，我们解释了 Move 包的基础知识：结构、清单文件、构建和测试流程。[在下一节中](./hello-sui.md)，我们将编写一个应用程序，了解 Move 代码如何构建以及这门语言能做什么。

## 进一步阅读

- [Manifest](./../concepts/manifest.md) 部分
- [The Move Reference](/reference/packages.html) 中的包相关内容
