# 存储功能

定义主要存储操作的模块是 `sui::transfer`。它在所有依赖[Sui 框架](./../programmability/sui-framework.md)的包中都是隐式导入的，因此，像其他隐式导入的模块（例如 `std::option` 或 `std::vector`）一样，不需要添加 `use` 声明。

## 概述

`transfer` 模块提供了执行所有三种与[所有权类型](./../object/ownership.md)匹配的存储操作的函数，我们之前已经解释过：

> 在本页面中，我们只讨论所谓的 _限制性_ 存储操作，稍后我们将介绍 _公共_ 操作，之后会引入 `store` 能力。

1. _Transfer_ - 将对象发送到地址，将其置于 _账户所有_ 状态;
2. _Share_ - 将对象置于 _共享_ 状态，因此可供所有人访问;
3. _Freeze_ - 将对象置于 _不可变_ 状态，因此成为公共常量，永远无法更改。

`transfer` 模块是大多数存储操作的首选，除了在下一章节中我们将讨论的 _动态字段_ 特殊情况。

## 所有权与引用：快速回顾

在[所有权与作用域](./../move-basics/ownership-and-scope.md)和[引用](./../move-basics/references.md)章节中，我们介绍了 Move 中所有权和引用的基础知识。当您使用存储函数时，理解这些概念非常重要。以下是最重要的几点回顾：

- Move 语义意味着值从一个作用域 _移动_ 到另一个作用域。换句话说，如果通过按值传递类型的实例给函数，它会 _移动_ 到函数作用域，并且在调用者作用域中无法访问。
- 要维护值的所有权，可以通过引用方式传递它。可以是 _不可变引用_ `&T` 或 _可变引用_ `&mut T`。然后该值被 _借用_，并且可以在调用者作用域中访问，但所有者保持不变。

```move
/// 通过值移动
public fun take<T>(value: T) { /* value 在此处被移动！ */ abort 0 }

/// 对于不可变引用
public fun borrow<T>(value: &T) { /* 在此处借用 value 可以读取 */ abort 0 }

/// 对于可变引用
public fun borrow_mut<T>(value: &mut T) { /* 在此处可变借用 value */ abort 0 }
```

<!-- TODO 部分：
    - 对象没有相关的存储类型
    - 相同类型的对象可以以不同方式存储
    - 必须在事务中通过它们的 ID 指定对象
 -->

## 转移

`transfer::transfer` 函数是用于将对象转移到另一个地址的公共函数。其签名如下，只接受具有 [`key` 能力](./key-ability.md) 和收件人地址的类型。请注意，对象通过值传递到函数中，因此它被移动到函数作用域，然后移动到接收者地址：

```move
// 文件：sui-framework/sources/transfer.move
public fun transfer<T: key>(obj: T, recipient: address);
```

在下面的示例中，您可以看到如何在定义并发送对象到事务发送者的模块中使用它。

```move
module book::transfer_to_sender {

    /// 具有 `key` 的结构体是一个对象。第一个字段是 `id: UID`！
    public struct AdminCap has key { id: UID }

    /// `init` 函数是发布模块时调用的特殊函数。这是创建应用对象的好地方。
    fun init(ctx: &mut TxContext) {
        // 在此作用域中创建一个新的 `AdminCap` 对象。
        let admin_cap = AdminCap { id: object::new(ctx) };

        // 将对象转移到事务发送者。
        transfer::transfer(admin_cap, ctx.sender());

        // admin_cap 已经消失！无法再访问它了。
    }

    /// 将 `AdminCap` 对象转移到 `recipient`。因此，接收者成为对象的所有者，只有他们可以访问它。
    public fun transfer_admin_cap(cap: AdminCap, recipient: address) {
        transfer::transfer(cap, recipient);
    }
}
```

当模块发布时，`init` 函数将被调用，并且我们在其中创建的 `AdminCap` 对象将被转移到事务发送者。`ctx.sender()` 函数返回当前事务的发送者地址。

一旦 `AdminCap` 被转移给发送者，例如 `0xa11ce`，发送者（仅发送者）将能够访问对象。对象现在是 _账户所有_ 的。

> 账户所有的对象受 _真正的所有权_ 影响 - 只有账户所有者才能访问它们。这是 Sui 存储模型中的基本概念。

让我们通过一个函数扩展示例，该函数使用 `AdminCap` 授权新对象的铸造并将其转移到另一个地址：

```move
/// 一些 `Gift` 对象，管理员可以 `铸造和转移`。
public struct Gift has key { id: UID }

/// 创建一个新的 `Gift` 对象并将其转移到 `recipient`。
public fun mint_and_transfer(
    _: &AdminCap, recipient: address, ctx: &mut TxContext
) {
    let gift = Gift { id: object::new(ctx) };
    transfer::transfer(gift, recipient);
}
```

`mint_and_transfer` 函数是一个公共函数，任何人都可以“可能”调用它，但是需要通过引用传递 `AdminCap` 对象作为第一个参数。如果没有它，函数将无法调用。这是限制访问特权功能的一种简单方式，称为 _[能力](./../programmability/capability.md)_。因为 `AdminCap` 对象是 _账户所有_ 的，只有 `0xa11ce` 才能调用 `mint_and_transfer` 函数。

发送给接收者的 `Gift` 将同样是 _账户所有_ 的，每个礼物都是独特的，且仅由接收者独占所有。

快速总结：

- `transfer` 函数用于将对象发送到地址；
- 对象变为 _账户所有_，只有接收者可以访问；
- 可以通过要求将对象作为参数传递来限制函数，从而创建 _能力_。

## 冻结

`transfer::freeze_object` 函数是用于将对象置于 _不可变_ 状态的公共函数。一旦对象被 _冻结_，它永远无法更改，且可以通过不可变引用被任何人访问。

该函数的签名如下，只接受具有 [`key` 能力](./key-ability.md) 的类型。与所有其他存储函数一样，它通过值接受对象：

```move
// 文件：sui-framework/sources/transfer.move
public fun freeze_object<T: key>(obj: T);
```

让我们扩展之前的示例，添加一个函数，允许管理员创建一个 `Config` 对象并将其冻结：

```move
/// 一些管理员可以 `创建和冻结` 的 `Config` 对象。
public struct Config has key {
    id: UID,
    message: String
}

/// 创建一个新的 `Config` 对象并将其冻结。
public fun create_and_freeze(
    _: &AdminCap,
    message: String,
    ctx: &mut TxContext
) {
    let config = Config

 {
        id: object::new(ctx),
        message
    };

    // 冻结对象，使其变为不可变。
    transfer::freeze_object(config);
}

/// 返回 `Config` 对象中的消息。
/// 可以通过不可变引用访问对象！
public fun message(c: &Config): String { c.message }
```

Config 是一个具有 `message` 字段的对象，`create_and_freeze` 函数创建一个新的 `Config` 并将其冻结。一旦对象被冻结，可以通过不可变引用被任何人访问。`message` 函数是一个公共函数，返回 `Config` 对象中的消息。Config 现在通过其 ID 公开可用，任何人都可以读取消息。

> 函数定义与对象的状态无关。可以定义一个接受可变引用的函数用于冻结对象。但是，它无法在冻结对象上调用。

`message` 函数可以在不可变的 `Config` 对象上调用，但是以下两个函数不能在冻结对象上调用：

```move
// === 以下函数不能在冻结对象上调用！ ===

/// 该函数可以定义，但不能在冻结对象上调用。
/// 仅允许不可变引用。
public fun message_mut(c: &mut Config): &mut String { &mut c.message }

/// 删除 `Config` 对象，按值接受它。
/// 不能在冻结对象上调用！
public fun delete_config(c: Config) {
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

## 后续步骤

现在您已经了解了 `transfer` 模块的主要功能，您可以开始在 Sui 上构建更复杂的涉及存储操作的应用程序。在下一章中，我们将介绍 [存储能力](./store-ability.md)，它允许在对象内部存储数据，并放宽我们在这里触及的转移限制。之后，我们将介绍 Sui 存储模型中最重要的类型 [UID 和 ID](./uid-and-id.md)。