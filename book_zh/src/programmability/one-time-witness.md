# 一次性见证（One Time Witness）

尽管常规的[见证（Witness）](./witness-pattern.md)是一种静态证明类型拥有权的好方法，但在某些情况下，我们需要确保见证仅被实例化一次。这就是一次性见证（One Time Witness，简称 OTW）的目的。

## 定义

一次性见证（OTW）是一种特殊类型的见证，只能使用一次。它不能手动创建，且每个模块中拥有唯一的实例。Sui 适配器将类型视为 OTW，如果满足以下规则：

1. 只具有 `drop` 能力。
2. 没有字段。
3. 不是泛型类型。
4. 模块名称为大写字母。

以下是 OTW 的示例：

```move
{{#include ../../../packages/samples/sources/programmability/one-time-witness.move:definition}}
```

OTW 不能手动构造，任何试图这样做的代码都会导致编译错误。OTW 可以作为[模块初始化器](./module-initializer.md)的第一个参数进行接收。由于 `init` 函数每个模块只调用一次，因此 OTW 保证只被实例化一次。

## 强制使用 OTW

要检查一个类型是否为 OTW，可以使用[Sui 框架](./sui-framework.md)的 `sui::types` 模块提供的特殊函数 `is_one_time_witness`。

```move
{{#include ../../../packages/samples/sources/programmability/one-time-witness.move:usage}}
```

<!-- ## 背景

在我们给出 OTW 的实际定义之前，让我们考虑一个简单的例子。我们想要构建一个 Coin 类型的通用实现，可以使用见证进行初始化。见证 `T` 的实例用于创建一个新的 `TreasuryCap<T>`，然后用于铸造一个新的 `Coin<T>`。

```move
module book::simple_coin {

    /// 控制 Coin 的供应。
    public struct TreasuryCap<phantom T> has key, store {
        id: UID,
        total_supply: u64,
    }

    /// Coin 类型，其中 `T` 是一个见证。
    public struct Coin<phantom T> has key, store {
        id: UID,
        value: u64,
    }

    /// 使用见证创建一个新的 TreasuryCap。
    /// 存在漏洞：我们可以使用相同的见证创建多个 TreasuryCap<T>。
    public fun new<T: drop>(_: T, ctx: &mut TxContext): TreasuryCap<T> {
        TreasuryCap { id: object::new(ctx), total_supply: 0 }
    }

    /// 我们使用普通见证来授权铸币。
    public fun mint<T>(
        treasury: &mut TreasuryCap<T>,
        value: u64,
        ctx: &mut TxContext
    ) {
        treasury.total_supply = treasury.total_supply + value;
        Coin { id: object::new(ctx), value }
    }
}
```

一个不诚实的开发者将能够使用相同的见证创建多个 `TreasuryCap`，并铸造出更多的 `Coin`。以下是这样一个恶意模块的示例：

```move
module book::simple_coin_cheater {
    /// Coin 见证。
    public struct Move has drop {}

    /// 使用 Move 见证初始化 TreasuryCap。
    /// ……并且做了两次！ >_<
    fun init(ctx: &mut TxContext) {
        let treasury_cap = book::simple_coin::new(Move {}, ctx);
        let secret_treasury = book::simple_coin::new(Move {}, ctx);

        transfer::public_transfer(treasury_cap, ctx.sender())
        transfer::public_transfer(secret_treasury, ctx.sender())
    }
}

```

上面的示例没有保护措施，以防止使用相同的见证发出多个 `TreasuryCap`，在实际应用中，这就产生了一个信任问题。如果是人为决策支持基于此实现的 Coin，则需要确保：

- 每个给定的 `T` 只有一个 `TreasuryCap`。
- 模块不能升级以发行更多的 `TreasuryCap`。
- 模块代码中没有任何用于发行更多 `TreasuryCap` 的后门。

然而，无法在 Move 代码中检查这些条件。为了避免对信任的需求，Sui 引入了 OTW 模式。

## 解决 Coin 问题

要解决多个 `TreasuryCap` 的问题，可以使用 OTW 模式。通过将 `COIN_OTW` 类型定义为 OTW，我们可以确保 `COIN_OTW` 仅使用一次。然后，可以使用 `COIN_OTW` 创建一个新的 `TreasuryCap` 并铸造一个新的 `Coin`。

```move
module book::coin_otw {

    /// `book::coin_otw` 模块的 OTW。
    struct COIN_OTW has drop {}

    /// 将 `COIN_OTW` 的实例作为第一个参数接收。
    fun init(otw: COIN_OTW, ctx: &mut TxContext) {
        let treasury_cap = book::simple_coin::new(COIN_OTW {}, ctx);
        transfer::public_transfer(treasury_cap, ctx.sender())
    }
}
```

 -->

<!-- ## 案例研究：Coin

TODO: 添加 TreasuryCap 和 Coin 背后的故事

-->

## 总结

OTW 模式是确保类型仅使用一次的强大工具。大多数开发者应该理解如何定义和接收 OTW，而 OTW 的检查和强制主要在库和框架中需要。例如，`sui::coin` 模块要求在 `coin::create_currency` 方法中使用 OTW，从而确保只创建一个 `coin::TreasuryCap`。

OTW 是为接下来我们将要介绍的[发布者（Publisher）](./publisher.md)对象打下基础的强大工具。

<!--

## 问题
- 还有其他方法可以防止多个 `TreasuryCap` 吗？
- 还有其他使用 OTW 的方法吗？

-->