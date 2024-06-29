# 关键能力

在[基本语法](./../move-basics)章节中，我们已经涵盖了四种能力中的两种 - [Drop](./../move-basics/drop-ability.md) 和 [Copy](./../move-basics/copy-ability.md)。它们影响值在作用域中的行为，并且与存储没有直接关系。现在是介绍 `key` 能力的时候了，这种能力允许结构体被存储。

历史上，`key` 能力被创建用来标记类型作为存储中的“关键”。具有 `key` 能力的类型可以在存储的顶层存储，并且可以被账户或地址直接拥有。随着[对象模型](./../object)的引入，`key` 能力自然成为对象的定义能力。

## 对象定义

具有 `key` 能力的结构体被认为是一个对象，并且可以在存储函数中使用。Sui 验证器要求结构体的第一个字段必须命名为 `id`，并且类型为 `UID`。

```move
public struct Object has key {
    id: UID, // 必需
    name: String,
}

/// 使用唯一ID创建一个新对象
public fun new(name: String, ctx: &mut TxContext): Object {
    Object {
        id: object::new(ctx), // 创建一个新的UID
        name,
    }
}
```

具有 `key` 能力的结构体仍然是一个结构体，可以拥有任意数量的字段和关联函数。对于打包、访问或解包结构体，并没有特殊的处理或语法要求。

然而，由于对象结构体的第一个字段必须是类型为 `UID` 的字段 - 一个不可复制也不可丢弃的类型（我们很快会深入讨论它！），因此结构体本身在设计上是不可丢弃的。

## 具有 `key` 能力的类型

由于具有 `key` 能力的类型需要 `UID`，因此 Move 中的原生类型和[标准库](./../move-basics/standard-library.md)中的类型都无法具有 `key` 能力。`key` 能力只存在于[Sui Framework](./../programmability/sui-framework.md)和自定义类型中。

## 下一步

关键能力定义了在 Move 中的对象，并且这些对象意图上是用来“存储”的。在下一节中，我们将介绍 `sui::transfer` 模块，该模块提供了对象的本地存储功能。

## 进一步阅读

- 在 Move 参考手册中的[类型能力](/reference/type-abilities.html)。

