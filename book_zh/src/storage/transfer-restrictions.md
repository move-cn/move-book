# 受限与公开转移

我们在[前面章节](./storage-functions.md)中描述的存储操作默认是受限的——它们只能在定义对象的模块中调用。
换句话说，类型必须是模块的 _内部_ 类型，才能用于存储操作。此限制由 Sui Verifier 实现，并在字节码层面强制执行。

然而，为了允许对象在其他模块中转移和存储，这些限制可以放宽。
`sui::transfer` 模块提供了一组 _public\_\*_ 函数，允许在其他模块中调用存储操作。
这些函数以 `public_` 为前缀，适用于所有模块和交易。

## 公开存储操作

`sui::transfer` 模块提供了以下公开函数。它们与我们之前讨论的函数几乎相同，但可以从任何模块中调用。

```move
// 文件: sui-framework/sources/transfer.move
/// `transfer` 函数的公开版本。
public fun public_transfer<T: key + store>(object: T, to: address) {}

/// `share_object` 函数的公开版本。
public fun public_share_object<T: key + store>(object: T) {}

/// `freeze_object` 函数的公开版本。
public fun public_freeze_object<T: key + store>(object: T) {}
```

为了说明这些函数的使用，考虑以下示例：
模块 A 定义了具有 `key` 能力的 ObjectK 和具有 `key + store` 能力的 ObjectKS，
而模块 B 尝试为这些对象实现一个 `transfer` 函数。

> 在此示例中，我们使用了 `transfer::transfer`，
> 但对于 `share_object` 和 `freeze_object` 函数，行为是相同的。

```move
/// 为 `ObjectK` 和 `ObjectKS` 分别定义 `key` 和 `key + store`的能力
module book::transfer_a {
    public struct ObjectK has key { id: UID }
    public struct ObjectKS has key, store { id: UID }
}

/// 从 `transfer_a` 导入 `ObjectK` 和 `ObjectKS` 类型，
/// 并尝试为它们实现不同的 `transfer` 函数
module book::transfer_b {
    // 类型并非此模块的内部类型
    use book::transfer_a::{ObjectK, ObjectKS};

    // 失败！ObjectK 没有 `store` 能力，并且 ObjectK 不是此模块的内部类型。
    public fun transfer_k(k: ObjectK, to: address) {
        sui::transfer::transfer(k, to);
    }

    // 失败！ObjectKS 具有 `store` 能力，但该函数不是公开的。
    public fun transfer_ks(ks: ObjectKS, to: address) {
        sui::transfer::transfer(ks, to);
    }

    // 失败！ObjectK 没有 `store` 能力，而 `public_transfer` 需要 `store` 能力。
    public fun public_transfer_k(k: ObjectK) {
        sui::transfer::public_transfer(k);
    }

    // 成功！ObjectKS 具有 `store` 能力，并且该函数是公开的。
    public fun public_transfer_ks(y: ObjectKS, to: address) {
        sui::transfer::public_transfer(y, to);
    }
}
```

进一步扩展上述示例：

- [ ] `transfer_k` 失败，因为 ObjectK 不是模块 `transfer_b` 的内部类型
- [ ] `transfer_ks` 失败，因为 ObjectKS 不是模块 `transfer_b` 的内部类型 
- [ ] `public_transfer_k` 失败，因为 ObjectK 没有 `store` 能力
- [x] `public_transfer_ks` 成功，因为 ObjectKS 具有 `store` 能力，并且转移是公开的

## `store` 能力的影响

是否为一个类型添加 `store` 能力需要谨慎决策。
一方面，`store` 能力实际上是让该类型可以被其他应用程序 _使用_ 的必要条件。
另一方面，它允许类型被 _封装_ 并改变原有的存储模型。
例如，一个角色可能是设计为由账户持有的，但如果具有 `store` 能力，该角色可以被冻结（无法被共享——此转换是受限的）。