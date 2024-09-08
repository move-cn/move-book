# 交易(Transaction)

交易是区块链世界中的一个基本概念，它是与区块链交互的方式。交易用于改变区块链的状态，也是唯一可以改变
状态的方式。在 Move 中，交易用于调用包中的函数、部署新的包或升级现有的包。

## 用户如何与程序交互

用户通过调用程序中的**公开函数**与区块链上的智能合约进行交互。这些公开函数定义了可以在交易中执行的操
作。交易是由账户发起的，账户发送交易时指定它要操作的对象。

## 交易结构

> 每个交易都会显式地指定它操作的对象！

交易由以下部分组成：

- **发送者**：签署交易的[账户](./what-is-an-account.md)；
- **指令列表**：要执行的操作链；
- **指令输入**：命令的参数，这些参数可以是`纯类型`（例如数字或字符串）或`对象类型`（交易需要访问的对
  象）；
- **Gas 对象**：支付交易费用的 `Coin` 对象；
- **Gas 价格与预算**：交易的费用。

## 输入

交易的输入是其参数，分为两种类型：

- **纯类型参数**：主要是[基础类型](../move-basics/primitive-types.html)，包括：

  - [`bool`](../move-basics/primitive-types.html#booleans)；
  - [无符号整数](../move-basics/primitive-types.html#integer-types) (`u8`, `u16`, `u32`, `u64`,
    `u128`, `u256`)；
  - [`address`](../move-basics/address.html)；
  - [`std::string::String`](../move-basics/string.html)，UTF8 字符串；
  - [`std::ascii::String`](../move-basics/string.html#ascii-strings)，ASCII 字符串；
  - [`vector<T>`](../move-basics/vector.html)，`T` 为纯类型；
  - [`std::option::Option<T>`](../move-basics/option.html)，`T` 为纯类型；
  - [`std::object::ID`](../storage/uid-and-id.html)，通常指向一个对象。详情见
    [什么是对象](../object/object-model.html)。

- **对象参数**：这些是交易需要访问的对象或对象的引用。对象参数必须是共享对象、冻结对象，或者是交易发
  送者拥有的对象，交易才能成功执行。更多信息请参见 [对象模型](../object/index.html)。

## 指令

Sui 交易可以包含多个指令。每个指令可以是一个内置命令（如发布包），也可以是对已发布包中的函数调用。指
令按顺序执行，它们可以使用之前指令的执行结果，形成一个链。交易要么整体成功，要么整体失败。


输入:

- sender = 0xa11ce

指令:

- payment = SplitCoins(Gas, [ 1000 ])
- item = MoveCall(0xAAA::market::purchase, [ payment ])
- TransferObjects(item, sender)

在这个例子中，交易由三个指令组成：

1. `SplitCoins`：一个内置命令，它从传入的对象（这里是 `Gas` 对象）中分割出新的硬币；
2. `MoveCall`：调用位于地址 `0xAAA` 的包中 `market` 模块的 `purchase` 函数，并传入 `payment` 对象作
   为参数；
3. `TransferObjects`：一个内置命令，将对象转移给接收者。

## 交易效果

交易效果是交易对区块链状态的改变。具体来说，交易可以通过以下方式改变状态：

- 使用 gas 对象支付交易费用
- 创建、更新或删除对象
- 触发事件

执行交易的结果包括多个部分：

- **交易摘要 (Transaction Digest)**：用于标识交易的哈希值；
- **交易数据 (Transaction Data)**：交易的输入、指令和 gas 对象；
- **交易效果 (Transaction Effect)**：交易的状态及其效果，具体包括：交易状态、对象更新及其新版本、使
  用的 gas 对象、交易的 gas 成本以及触发的事件；
- **事件 (Events)**：交易触发的自定义[事件](./../programmability/events.md)；
- **对象变更 (Object Changes)**：对象的变更，包括所有权的变化；
- **余额变更 (Balance Changes)**：账户余额的变化。
