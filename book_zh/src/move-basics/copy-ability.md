# 能力：复制

在 Move 中，类型上的 _copy_ 能力表示该类型的实例或值可以被复制。尽管在处理数字或其他简单类型时，这种行为可能非常自然，但在 Move 中自定义类型默认不具备这种能力。这是因为 Move 旨在表达数字资产和资源，而无法复制是资源模型的一个关键要素。

然而，Move 类型系统允许您定义具有 _copy_ 能力的自定义类型。

```move
{{#include ../../../packages/samples/sources/move-basics/copy-ability.move:copyable}}
```

在上面的例子中，我们定义了一个具有 _copy_ 能力的自定义类型 `Copyable`。这意味着 `Copyable` 的实例可以被复制，无论是隐式的还是显式的。

```move
{{#include ../../../packages/samples/sources/move-basics/copy-ability.move:copyable_test}}
```

在上面的例子中，`a` 被隐式地复制到 `b`，然后使用解引用操作符显式地复制到 `c`。如果 `Copyable` 没有 _copy_ 能力，代码将无法编译，Move 编译器会抛出错误。

## 复制与丢弃

`copy` 能力与 [`drop` 能力](./drop-ability.md) 密切相关。如果一个类型具有 _copy_ 能力，那么它很可能也应该具有 `drop` 能力。这是因为 _drop_ 能力用于在实例不再需要时清理资源。如果一个类型只有 _copy_，那么管理它的实例会变得更加复杂，因为这些值不能被忽略。

```move
{{#include ../../../packages/samples/sources/move-basics/copy-ability.move:copy_drop}}
```

Move 中的所有原始类型都表现得像是具有 _copy_ 和 _drop_ 能力。这意味着它们可以被复制和丢弃，并且 Move 编译器会为它们处理内存管理。

## 具有 `copy` 能力的类型

Move 中的所有本机类型都具有 `copy` 能力。这包括：

- [布尔值](./../move-basics/primitive-types.md#booleans)
- [无符号整数](./../move-basics/primitive-types.md#integer-types)
- [向量](./../move-basics/vector.md)
- [地址](./../move-basics/address.md)

标准库中定义的所有类型也具有 `copy` 能力。这包括：

- [Option](./../move-basics/option.md)
- [String](./../move-basics/string.md)
- [TypeName](./../move-basics/type-reflection.md#typename)

## 进一步阅读

- [类型能力](/reference/type-abilities.html) 在 Move 参考中。