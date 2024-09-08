# 模式：烫手山芋

在能力系统中，没有任何能力的结构体被称为 _烫手山芋_ 。
它不能被存储（既不能作为[对象](./../storage/key-ability.md)，
也不能作为[另一个结构体的字段](./../storage/store-ability.md)），
也不能被[复制](./../move-basics/copy-ability.md)或[丢弃](./../move-basics/drop-ability.md)。
因此，一旦构造完成，它必须优雅地[由其模块解包](./../move-basics/struct.md)，
否则由于未使用的值无法丢弃，交易将中止。

> 如果你熟悉支持 _回调_ 的编程语言，可以将“烫手山芋”理解为必须调用回调函数的义务。如果你不调用它，交易将中止。

这个名字源自儿童游戏“烫手山芋”，游戏中球在玩家之间快速传递，没有人希望在音乐停止时成为最后一个持球的人，否则他们会出局。
这正是该模式的最佳比喻——“烫手山芋”结构体的实例在函数调用之间传递，任何模块都不能保留它。

## 定义一个烫手山芋

没有任何能力的结构体都可以是“烫手山芋”。例如，以下结构体就是一个“烫手山芋”：

```move
{{#include ../../../packages/samples/sources/programmability/hot-potato-pattern.move:definition}}
```

由于 `Request` 没有任何能力，不能被存储或忽略，因此模块必须提供一个解包它的函数。例如：

```move
{{#include ../../../packages/samples/sources/programmability/hot-potato-pattern.move:new_request}}
```

## 使用示例

在以下示例中，`Promise` 作为“烫手山芋”被用来确保从容器中借出的值在使用后被归还。
`Promise` 结构体包含了被借出对象的 ID 和容器的 ID，确保借出的值没有被替换成其他对象，
并且被正确归还到对应的容器中。

```move
{{#include ../../../packages/samples/sources/programmability/hot-potato-pattern.move:container_borrow}}
```

## 应用场景

以下是“烫手山芋”模式的一些常见应用场景：

### 借用

如[上述示例](#example-usage)所示，
“烫手山芋”模式在借用场景中非常有效，能够保证借出的值被正确归还到原始容器中。
虽然示例中聚焦于存储在 `Option` 中的值，
但同样的模式也可以应用于其他存储类型，比如[动态字段](./dynamic-fields.md)。

### 闪电贷

“烫手山芋”模式的经典示例是闪电贷。闪电贷是一种在同一笔交易中借入并偿还的贷款。
借入的资金用于执行一些操作，之后归还给贷款方。
“烫手山芋”模式确保借入的资金会被归还给贷款方。

此模式的使用示例可能如下所示：

```move
// 从贷款方借入资金。
let (asset_a, potato) = lender.borrow(amount);

// 使用借入的资金执行一些操作。
let asset_b = dex.trade(loan);
let proceeds = another_contract::do_something(asset_b);

// 保留佣金并将其余部分返还给贷款方。
let pay_back = proceeds.split(amount, ctx);
lender.repay(pay_back, potato);
transfer::public_transfer(proceeds, ctx.sender());
```

### 变量路径执行

“烫手山芋”模式可以用于引入执行路径中的变化。
例如，如果某个模块允许用“奖励积分”或美元购买一部 `Phone`，那么可以使用“烫手山芋”模式将购买与支付解耦。
这种方式与某些商店的工作方式非常相似——你从货架上取下商品，然后到收银台付款。

```move
{{#include ../../../packages/samples/sources/programmability/hot-potato-pattern.move:phone_shop}}
```

这种解耦技术允许将购买逻辑与支付逻辑分离，使代码更加模块化并更易于维护。
`Ticket` 可以拆分为一个独立的模块，提供基础的支付接口，而商店的实现可以扩展以支持其他商品，而无需更改支付逻辑。

### 组合模式

“烫手山芋”模式可以用于以组合的方式将不同模块连接在一起。
模块可以定义与“烫手山芋”交互的方式，例如给它盖上类型签名，或从中提取一些信息。
这样，“烫手山芋”可以在不同模块之间传递，甚至在同一交易中传递到不同的包中。

最重要的组合模式是[请求模式](./request-pattern.md)，我们将在下一节中介绍。

### 在 Sui 框架中的应用

该模式在 Sui 框架中以各种形式使用。以下是一些示例：

- `sui::borrow` - 使用“烫手山芋”模式确保借出的值被正确归还到原始容器。
- `sui::transfer_policy` - 定义了 `TransferRequest`，一种只能在满足所有条件时才能被消耗的“烫手山芋”。
- `sui::token` - 在闭环代币系统中，`ActionRequest` 携带已执行操作的信息，并像 `TransferRequest` 一样收集批准。

## 总结

- “烫手山芋”是没有能力的结构体，它必须伴随有创建和销毁的方法。
- “烫手山芋”用于确保在交易结束前执行某些操作，类似于回调函数。
- “烫手山芋”最常见的应用场景包括借用、闪电贷、变量路径执行和组合模式。
