# 事务上下文

每个事务都有执行上下文。上下文是在执行过程中程序可以访问的一组预定义变量。例如，每个事务都有一个发送者地址，事务上下文包含一个保存发送者地址的变量。

事务上下文可以通过`TxContext`结构体在程序中访问。该结构体定义在`sui::tx_context`模块中，包含以下字段：

```move
// File: sui-framework/sources/tx_context.move
/// Information about the transaction currently being executed.
/// This cannot be constructed by a transaction--it is a privileged object created by
/// the VM and passed in to the entrypoint of the transaction as `&mut TxContext`.
struct TxContext has drop {
    /// The address of the user that signed the current transaction
    sender: address,
    /// Hash of the current transaction
    tx_hash: vector<u8>,
    /// The current epoch number
    epoch: u64,
    /// Timestamp that the epoch started at
    epoch_timestamp_ms: u64,
    /// Counter recording the number of fresh id's created while executing
    /// this transaction. Always 0 at the start of a transaction
    ids_created: u64
}
```

事务上下文不能手动构造或直接修改。它由系统创建，并作为引用传递给事务中的函数。在[Transaction](./../concepts/what-is-a-transaction.md)中调用的任何函数都可以访问上下文并将其传递给嵌套调用。

> `TxContext`必须是函数签名中的最后一个参数。

## 读取事务上下文

除了`ids_created`字段外，`TxContext`中的所有字段都有对应的getter方法。这些getter方法在`sui::tx_context`模块中定义，并对程序可用。这些getter方法不需要`&mut`，因为它们不修改上下文。

```move
{{#include ../../../packages/samples/sources/programmability/transaction-context.move:reading}}
```

## 可变性

`TxContext`用于在系统中创建新对象（或仅仅是`UID`）。新的UID是从事务摘要派生的，为了使摘要唯一，需要一个变化的参数。Sui使用`ids_created`字段来实现这一点。每次创建新的UID时，`ids_created`字段会增加一。这样，摘要始终是唯一的。

在内部，它由`derive_id`函数表示：

```move
// File: sui-framework/sources/tx_context.move
native fun derive_id(tx_hash: vector<u8>, ids_created: u64): address;
```

## 生成唯一地址

底层的`derive_id`函数也可以在你的程序中用于生成唯一地址。该函数本身没有暴露出来，但在`sui::tx_context`模块中提供了一个包装函数`fresh_object_address`。如果你需要在程序中生成唯一标识符，它可能会很有用。

```move
// File: sui-framework/sources/tx_context.move
/// Create an `address` that has not been used. As it is an object address, it will never
/// occur as the address for a user.
/// In other words, the generated address is a globally unique object ID.
public fun fresh_object_address(ctx: &mut TxContext): address {
    let ids_created = ctx.ids_created;
    let id = derive_id(*&ctx.tx_hash, ids_created);
    ctx.ids_created = ids_created + 1;
    id
}
```