# 模式：证明者

证明者是通过构建证据来证明存在的一种模式。在编程的背景下，证明者是通过提供一个只有在属性成立时才能构建的值来证明系统的某个属性。

## 在 Move 中的证明者模式

在 [结构体](./../move-basics/struct.md) 部分中，我们展示了结构体只能由定义它的模块创建或打包。因此，在 Move 中，模块通过构建类型来证明对其的所有权。这是 Move 中最重要的模式之一，广泛用于泛型类型的实例化和授权。

从实际角度来看，为了使用证明者，必须有一个期望证明者作为参数的函数。在下面的示例中，`new` 函数期望一个 `T` 类型的证明者来创建 `Instance<T>` 实例。

> 通常情况下，证明者结构体不会被存储，因此函数可能需要该类型的 [Drop](./../move-basics/drop-ability.md) 能力。

```move
module book::witness {
    /// 需要证明者才能创建的结构体。
    public struct Instance<T> { t: T }

    /// 使用提供的 T 创建 `Instance<T>` 的新实例。
    public fun new<T>(witness: T): Instance<T> {
        Instance { t: witness }
    }
}
```

构造 `Instance<T>` 的唯一方法是调用 `new` 函数，并提供类型 `T` 的一个实例。这是 Move 中证明者模式的基本示例。提供证明者的模块通常会有相应的实现，例如下面的 `book::witness_source` 模块：

```move
module book::witness_source {
    use book::witness::{Self, Instance};

    /// 作为证明者使用的结构体。
    public struct W {}

    /// 创建 `Instance<W>` 的新实例。
    public fun new_instance(): Instance<W> {
        witness::new(W {})
    }
}
```

将结构体 `W` 的实例传递给 `new_instance` 函数可以创建一个 `Instance<W>`，从而证明了模块 `book::witness_source` 拥有类型 `W`。

## 实例化泛型类型

证明者允许使用具体类型实例化泛型类型。这对于从类型继承关联行为，并有选择地扩展这些行为（如果模块提供了这样的能力）非常有用。

```move
// 文件：sui-framework/sources/balance.move
/// 供应类型为 T。用于铸造和销毁。
public struct Supply<phantom T> has key, store {
    id: UID,
    value: u64
}

/// 使用提供的证明者为类型 T 创建新的供应。
public fun create_supply<T: drop>(_w: T): Supply<T> {
    Supply { value: 0 }
}

/// 获取 `Supply` 的值。
public fun supply_value<T>(supply: &Supply<T>): u64 {
    supply.value
}
```

上面的示例来自于 [Sui Framework](./sui-framework.md) 的 `balance` 模块，其中 `Supply` 是一个泛型结构体，只能通过提供类型 `T` 的证明者来构建。证明者是按值获取并且被丢弃 - 因此 `T` 必须具有 [drop](./../move-basics/drop-ability.md) 能力。

然后可以使用实例化的 `Supply<T>` 来铸造新的 `Balance<T>`，其中 `T` 是供应的类型。

```move
// 文件：sui-framework/sources/balance.move
/// 可存储的余额。
struct Balance<phantom T> has store {
    value: u64
}

/// 增加供应量 `value` 并创建具有此值的新 `Balance<T>`。
public fun increase_supply<T>(self: &mut Supply<T>, value: u64): Balance<T> {
    assert!(value < (18446744073709551615u64 - self.value), EOverflow);
    self.value = self.value + value;
    Balance { value }
}
```

## 一次性证明者

虽然结构体可以任意次数地创建，但有些情况下需要确保结构体只能创建一次。为此，Sui 提供了“一次性证明者” - 一种特殊的证明者，只能使用一次。我们将在[下一节](./one-time-witness.md)中详细解释。

## 总结

- 证明者是通过构建证据来证明某个属性的模式。
- 在 Move 中，通过构建类型来证明模块对其的所有权。
- 证明者经常用于泛型类型的实例化和授权。

## 下一步

在下一节中，我们将学习关于[一次性证明者](./one-time-witness.md)模式。