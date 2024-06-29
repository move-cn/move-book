# 包

Move 是一种用于编写智能合约的语言，这些合约被存储和运行在区块链上。一个程序通常被组织成一个包。包被发布到区块链上，并通过一个 [地址](./address.md) 进行标识。已发布的包可以通过发送[交易](./what-is-a-transaction.md)来调用其函数进行交互。它还可以作为其他包的依赖项。

> 要创建一个新的包，请使用 `sui move new` 命令。要了解更多命令的详细信息，请运行 `sui move new --help`。

包由模块组成 - 单独的作用域，包含函数、类型和其他项。

```
package 0x...
    module a
        struct A1
        fun hello_world()
    module b
        struct B1
        fun hello_package()
```

## 包结构

在本地，一个包是一个包含 `Move.toml` 文件和一个 `sources` 目录的文件夹。`Move.toml` 文件 - 称为 "包清单" - 包含关于包的元数据，而 `sources` 目录包含模块的源代码。包通常具有以下结构：

```
sources/
    my_module.move
    another_module.move
    ...
tests/
    ...
examples/
    using_my_module.move
Move.toml
```

`tests` 目录是可选的，包含包的测试代码。放置在 `tests` 目录中的代码不会发布到链上，仅在测试中使用。`examples` 目录可以用于包含代码示例，同样也不会发布到链上。

## 已发布的包

在开发过程中，包没有地址，需要设置为 `0x0`。一旦包被发布，它将在区块链上获得一个唯一的 [地址](./address.md)，其中包含其模块的字节码。已发布的包变得 _不可变_，可以通过发送交易进行交互。

```
0x...
    my_module: <bytecode>
    another_module: <bytecode>
```

## 链接

- [包清单](./manifest.md)
- [地址](./address.md)
- 在 Move 参考中的 [Packages](/reference/packages.html) 页面