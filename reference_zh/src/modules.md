# Modules(模块)

**模块** 是定义类型及操作这些类型的函数的核心程序单元。结构类型定义了 Move 存储的模式，而模块函数定义了与这些类型的值交互的规则。虽然模块本身也存储在存储中，但在 Move 程序内部是不可访问的。在区块链环境中，模块存储在链上，通常称为 "发布" 过程。在发布后，可以根据特定 Move 实例的规则调用 [`entry`](./functions.md#entry-modifier) 和 [`public`](./functions.md#visibility) 函数。

## 语法

模块具有以下语法：

```text
module <address>::<identifier> {
    (<use> | <type> | <function> | <constant>)*
}
```

其中 `<address>` 是指定模块所属包的有效 [地址](./primitive-types/address.md)。

例如：

```move
module 0x42::test {
    public struct Example has copy, drop { i: u64 }

    use std::debug;

    const ONE: u64 = 1;

    public fun print(x: u64) {
        let sum = x + ONE;
        let example = Example { i: sum };
        debug::print(&sum)
    }
}
```

## 名称

`module test_addr::test` 部分指定了模块 `test` 将在名称为 `test_addr` 的包设置中分配的数字 [地址](./primitive-types/address.md) 值下进行发布。

通常应使用 [命名地址](./primitive-types/address.md) 来声明模块（而不是直接使用数字值）。例如：

```move
module test_addr::test {
    public struct Example has copy, drop { a: address }

    friend test_addr::another_test;

    public fun print() {
        let example = Example { a: @test_addr };
        debug::print(&example)
    }
}
```

这些命名地址通常与 [包](./packages.md) 的名称匹配。

因为命名地址仅存在于源语言级别和编译过程中，在字节码级别上，命名地址将完全替换为其值。例如，如果我们有以下代码：

```move
fun example() {
    my_addr::m::foo(@my_addr);
}
```

并且将其编译时设置 `my_addr` 设置为 `0xC0FFEE`，则其操作上等效于：

```move
fun example() {
    0xC0FFEE::m::foo(@0xC0FFEE);
}
```

尽管在源级别上这两种访问方式是等效的，但最佳实践是始终使用命名地址而不是分配给该地址的数字值。

模块名可以以小写字母 `a` 到 `z` 或大写字母 `A` 到 `Z` 开始。在第一个字符之后，模块名可以包含下划线 `_`、字母 `a` 到 `z`、字母 `A` 到 `Z` 或数字 `0` 到 `9`。

```move
module a::my_module {}
module a::foo_bar_42 {}
```

通常，模块名以小写字母开头。名为 `my_module` 的模块应存储在名为 `my_module.move` 的源文件中。

## 成员

模块块内的所有成员可以以任何顺序出现。基本上，模块是 [`types`](./structs.md) 和 [`functions`](./functions.md) 的集合。[`use`](./uses.md) 关键字用于引用其他模块的成员。[`const`](./constants.md) 关键字定义可以在模块函数中使用的常量。

[`friend`](./friends.md) 语法是一种已废弃的概念，用于指定一组受信任的模块列表。该概念已被 [`public(package)`](./functions.md#visibility) 取代。

<!-- TODO 成员访问规则 -->