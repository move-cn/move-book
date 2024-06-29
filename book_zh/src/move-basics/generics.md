# 泛型

泛型是一种定义可以适用于任何类型的类型或函数的方式。当你希望编写一个可以与不同类型一起使用的函数，或者当你希望定义一个可以容纳任何其他类型的类型时，泛型就非常有用。泛型是Move中许多高级功能的基础，例如集合、抽象实现等。

## 标准库中的泛型

在本章中，我们已经提到了 [vector](./vector.md) 类型，它是一个可以容纳任何其他类型的泛型类型。标准库中另一个泛型类型的例子是 [Option](./option.md) 类型，它用于表示可能存在或不存在的值。

## 泛型语法

要定义一个泛型类型或函数，类型签名必须具有一个用尖括号（`<`和`>`）括起来的泛型参数列表。泛型参数之间用逗号分隔。

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:container}}
```

在上面的示例中，`Container` 是一个具有单个类型参数 `T` 的泛型类型，容器的 `value` 字段存储了 `T` 类型的值。`new` 函数是一个具有单个类型参数 `T` 的泛型函数，它返回一个带有给定值的 `Container`。泛型类型必须使用具体类型进行初始化，而泛型函数必须使用具体类型进行调用。

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:test_container}}
```

在测试函数 `test_generic` 中，我们展示了三种等效的方法来创建一个具有 `u8` 值的新 `Container`。由于数字类型需要进行推断，我们指定了数字字面量的类型。

## 多个类型参数

可以定义具有多个类型参数的类型或函数。类型参数之间用逗号分隔。

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:pair}}
```

在上面的示例中，`Pair` 是一个具有两个类型参数 `T` 和 `U` 的泛型类型，`new_pair` 函数是一个具有两个类型参数 `T` 和 `U` 的泛型函数。该函数返回一个带有给定值的 `Pair`。类型参数的顺序很重要，它应该与类型签名中的类型参数顺序匹配。

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:test_pair}}
```

如果我们在 `new_pair` 函数中添加了另一个实例，交换类型参数，并尝试比较两个类型，我们会发现类型签名是不同的，无法进行比较。

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:test_pair_swap}}
```

变量 `pair1` 和 `pair2` 的类型是不同的，因此无法进行比较。

## 为什么使用泛型？

在上面的示例中，我们专注于实例化泛型类型和调用泛型函数，以创建这些类型的实例。然而，泛型的真正威力在于能够为基础泛型类型定义共享行为，然后独立于具体类型使用它。这在处理集合、抽象实现和其他Move中的高级功能时特别有用。

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:user}}
```

在上面的示例中，`User` 是一个具有单个类型参数 `T` 的泛型类型，具有共享的 `name` 和 `age` 字段，以及可以存储任何类型的泛型 `metadata` 字段。无论 `metadata` 是什么，`User` 的所有实例都具有相同的字段和方法。

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:update_user}}
```

##虚拟类型参数

在某些情况下，您可能希望定义一个具有未在类型的字段或方法中使用的类型参数的泛型类型。这被称为 _虚拟类型参数_。当您希望定义一个可以容纳任何其他类型的类型，但希望对类型参数施加一些约束时，虚拟类型参数非常有用。

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:phantom}}
```

这里的 `Coin` 类型不包含使用类型参数 `T` 的字段或方法。它用于区分不同类型的硬币，并对类型参数 `T` 施加一些约束。

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:test_phantom}}
```

在上面的示例中，我们演示了如何创建具有不同虚拟类型参数 `USD` 和 `EUR` 的两个不同的 `Coin` 实例。类型参数 `T` 在 `Coin` 类型的字段或方法中没有使用，但它用于区分不同类型的硬币。它将确保 `USD` 和 `EUR` 硬币不会混淆。

## 对类型参数的约束

类型参数可以限制为具有某些功能。当您需要内部类型允许某些行为（例如 _复制_ 或 _丢弃_）时，这非常有用。约束类型参数的语法是 `T: <ability> + <ability>`。

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:constraints}}
```

Move 编译器将强制要求类型参数 `T` 具有指定的功能。如果类型参数不具备指定的功能，代码将无法编译。

<!-- TODO: failure case -->

```move
{{#include ../../../packages/samples/sources/move-basics/generics.move:test_constraints}}
```

## 进一步阅读

- [泛型](/reference/generics.html) 在 Move 参考文档中。