```move
) {
    let Config { id, message: _ } = c;
    id.delete();
}
```

总结：

- `transfer::freeze_object` 函数用于将对象置于 _不可变_ 状态；
- 一旦对象被冻结，它永远无法更改、删除或转移，可以通过不可变引用被任何人访问；

## 账户拥有 -> 冻结

由于 `transfer::freeze_object` 的签名接受任何具有 `key` 能力的类型，它可以接受在同一作用域中创建的对象，但也可以接受账户拥有的对象。这意味着 `freeze_object` 函数可以用来冻结转移给发送者的对象。出于安全考虑，我们不希望冻结 `AdminCap` 对象——允许任何人访问它是一个安全风险。然而，我们可以冻结铸造并转移给接收者的 `Gift` 对象：

> 单一所有者 -> 不可变转换是可能的！

```move
/// 冻结 `Gift` 对象，使其变为不可变。
public fun freeze_gift(gift: Gift) {
    transfer::freeze_object(gift);
}
```

## 共享

`transfer::share_object` 函数是用于将对象置于 _共享_ 状态的公共函数。一旦对象被 _共享_，它可以通过可变引用（当然，也可以通过不可变引用）被任何人访问。该函数的签名如下，只接受具有 [`key` 能力](./key-ability.md) 的类型：

```move
// 文件：sui-framework/sources/transfer.move
public fun share_object<T: key>(obj: T);
```

一旦对象被 _共享_，它可以通过可变引用公开访问。

## 特殊情况：共享对象删除

虽然共享对象通常不能按值获取，但有一种特殊情况可以——如果获取它的函数删除对象。这是 Sui 存储模型中的一个特殊情况，用于允许删除共享对象。为了展示它如何工作，我们将创建一个函数，创建并共享一个 Config 对象，然后再创建一个删除它的函数：

```move
/// 创建一个新的 `Config` 对象并共享它。
public fun create_and_share(message: String, ctx: &mut TxContext) {
    let config = Config {
        id: object::new(ctx),
        message
    };

    // 共享对象，使其变为共享。
    transfer::share_object(config);
}
```

`create_and_share` 函数创建一个新的 `Config` 对象并共享它。对象现在可以通过可变引用公开访问。让我们创建一个删除共享对象的函数：

```move
/// 删除 `Config` 对象，按值获取它。
/// 可以在共享对象上调用！
public fun delete_config(c: Config) {
    let Config { id, message: _ } = c;
    id.delete();
}
```

`delete_config` 函数按值获取 `Config` 对象并删除它，Sui 验证器会允许此调用。然而，如果函数返回 `Config` 对象或试图 `freeze` 或 `transfer` 它，Sui 验证器将拒绝事务。

```move
// 不会成功！
public fun transfer_shared(c: Config, to: address) {
    transfer::transfer(c, to);
}
```

总结：

- `share_object` 函数用于将对象置于 _共享_ 状态；
- 一旦对象被共享，它可以通过可变引用被任何人访问；
- 共享对象可以删除，但不能转移或冻结。

## 公共转移与限制转移

我们在[之前的章节](./storage-functions.md)中描述的存储操作默认是受限制的——它们只能在定义对象的模块中调用。换句话说，类型必须是模块内部才能在存储操作中使用。这种限制是在 Sui 验证器中实现的，并在字节码级别执行。

然而，为了允许对象在其他模块中被转移和存储，这些限制可以被放宽。`sui::transfer` 模块提供了一组 _public\_\*_ 函数，允许在其他模块中调用存储操作。这些函数以 `public_` 为前缀，并且对所有模块和交易都是可用的。

## 公共存储操作

`sui::transfer` 模块提供了以下公共函数。它们几乎与我们已经介绍的函数相同，但可以从任何模块调用。

```move
// 文件：sui-framework/sources/transfer.move
/// `transfer` 函数的公共版本。
public fun public_transfer<T: key + store>(object: T, to: address) {}

/// `share_object` 函数的公共版本。
public fun public_share_object<T: key + store>(object: T) {}

/// `freeze_object` 函数的公共版本。
public fun public_freeze_object<T: key + store>(object: T) {}
```

为了说明这些函数的使用，考虑以下示例：模块 A 定义了具有 `key` 和具有 `key + store` 能力的 ObjectK 和 ObjectKS，并且模块 B 尝试为这些对象实现 `transfer` 函数。

> 在这个示例中，我们使用 `transfer::transfer`，但是对于 `share_object` 和 `freeze_object` 函数，行为是相同的。

```move
/// 定义具有 `key` 和 `key + store` 能力的 `ObjectK` 和 `ObjectKS`
/// 分别
module book::transfer_a {
    public struct ObjectK has key { id: UID }
    public struct ObjectKS has key, store { id: UID }
}

/// 导入 `ObjectK` 和 `ObjectKS` 类型自 `transfer_a` 并尝试实现
/// 为它们的不同 `transfer` 函数
module book::transfer_b {
    // types are not internal to this module
    use book::transfer_a::{ObjectK, ObjectKS};

    // 失败！ObjectK 不是 `store`，且 ObjectK 不是该模块的内部类型
    public fun transfer_k(k: ObjectK, to: address) {
        sui::transfer::transfer(k, to);
    }

    // 失败！ObjectKS 具有 `store`，但该函数不是公共的
    public fun transfer_ks(ks: ObjectKS, to: address) {
        sui::transfer::transfer(ks, to);
    }

    // 失败！ObjectK 不具有 `store`，`public_transfer` 需要 `store`
    public fun public_transfer_k(k: ObjectK) {
        sui::transfer::public_transfer(k);
    }

    // 成功！ObjectKS 具有 `store`，并且函数是公共的
    public fun public_transfer_ks(y: ObjectKS, to: address) {
        sui::transfer::public_transfer(y, to);
    }
}
```

对上述示例进行扩展：

- [ ] `transfer_k` 失败，因为 ObjectK 不是模块 `transfer_b` 的内部类型
- [ ] `transfer_ks` 失败，因为 ObjectKS 不是模块 `transfer_b` 的内部类型
- [ ] `public_transfer_k` 失败，因为 ObjectK 没有 `store` 能力
- [x] `public_transfer_ks` 成功，因为 ObjectKS 具有 `store` 能力，并且转移是公共的

## `store` 能力的影响

是否为类型添加 `store` 能力的决定应谨慎进行。一方面，它是使该类型能够被其他应用程序使用的实际要求。另一方面，它允许 _包装_ 和更改预期的存储模型。例如，一个角色可能被设计为由账户拥有，但具有 `store` 能力，它可以被冻结（不能

被共享 - 这种转换受限制）