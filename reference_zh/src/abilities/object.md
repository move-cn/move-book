# Sui 对象

在 Sui 中，`key` 用于表示一个 _对象_。对象是 Sui 中唯一存储数据的方式，允许数据在交易之间持久化。

更多详细信息，请参阅 Sui 文档：

- [对象模型](https://docs.sui.io/concepts/object-model)
- [Move 对象规则](https://docs.sui.io/concepts/sui-move-concepts#global-unique)
- [对象转移](https://docs.sui.io/concepts/transfers)

## 对象规则

对象是具有 [`key`](../abilities.md#key) 能力的 [`struct`](../structs.md)。struct 的第一个字段必须是 `id: sui::object::UID`。这个 32 字节的字段（是一个 [`address`](../primitive-types/address.md) 的强类型封装）用于唯一标识对象。

请注意，由于 `sui::object::UID` 只有 `store` 能力（没有 `copy` 或 `drop`），因此任何对象都没有 `copy` 或 `drop` 能力。

## 转移规则

对象的所有权可以在 `sui::transfer` 模块中改变和转移。该模块中的许多函数都有“public”和“private”变体，其中“private”变体只能在定义对象类型的模块内部调用。而“public”变体只有在对象具有 `store` 能力时才能调用。

例如，如果我们在模块 `my_module` 中定义了两个对象 `A` 和 `B`：

```move
module a::my_module {
    public struct A has key {
        id: sui::object::UID,
    }
    public struct B has key, store {
        id: sui::object::UID,
    }
}
```

`A` 只能在 `a::my_module` 内部使用 `sui::transfer::transfer` 进行转移，而 `B` 可以在任何地方使用 `sui::transfer::public_transfer` 进行转移。这些规则由 Sui 的自定义类型系统（字节码验证器）规则强制执行。