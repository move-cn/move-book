# 能力：Store

现在你已经了解了通过[`key`](./key-ability.md)能力启用的顶层存储函数，我们可以讨论列表中的最后一个能力 - `store`。

## 定义

`store`是一种特殊的能力，允许将类型存储在对象中。该能力是字段可以在具有`key`能力的结构体中使用的必需条件。换句话说，`store`能力允许值被包装在对象中。

> `store`能力还放宽了转移操作的限制。我们在[受限和公共转移](./transfer-restrictions.md)部分中会详细讨论。

## 示例

在之前的章节中，我们已经使用了具有`key`能力的类型：所有对象必须有一个`UID`字段，我们在示例中使用了它；我们还将`Storable`类型作为`Config`结构体的一部分使用。`Config`类型也具有`store`能力。

```move
/// 这个类型具有`store`能力。
public struct Storable has store {}

/// Config 包含一个具有`store`能力的`Storable`字段。
public struct Config has key, store {
    id: UID,
    stores: Storable,
}

/// MegaConfig 包含一个具有`store`能力的`Config`字段。
public struct MegaConfig has key {
    id: UID,
    config: Config, // 在这里！
}
```

## 具有`store`能力的类型

在Move中，除了引用之外，所有原生类型都具有`store`能力。包括：

- [bool](./../move-basics/primitive-types.md#booleans)
- [无符号整数](./../move-basics/primitive-types.md#integer-types)
- [vector](./../move-basics/vector.md)
- [address](./../move-basics/address.md)

标准库中定义的所有类型也具有`store`能力。包括：

- [Option](./../move-basics/option.md)
- [String](./../move-basics/string.md)
- [TypeName](./../move-basics/type-reflection.md#typename)

## 进一步阅读

- Move参考中的[类型能力](/reference/type-abilities.html)。