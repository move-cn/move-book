# Sui Framework

Sui Framework 是 [Package Manifest](./../concepts/manifest.md) 中的默认依赖集。它依赖于 [标准库](./../move-basics/standard-library.md)，提供了特定于 Sui 的功能，包括与存储的交互，以及 Sui 特定的本地类型和模块。

为方便起见，我们将 Sui Framework 中的模块分为多个类别。但它们仍然属于同一个框架。

## 核心模块

<div class="modules-table">

| 模块                                                                                           | 描述                                                                 | 章节                                          |
| ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ | --------------------------------------------- |
| [sui::address](https://docs.sui.io/references/framework/sui-framework/address)                   | 添加了对 [地址类型](./../move-basics/address.md) 的转换方法       | [地址](./../move-basics/address.md)            |
| [sui::transfer](https://docs.sui.io/references/framework/sui-framework/transfer)                 | 实现了对象的存储操作                                               | [它以一个对象开始](./../object)                 |
| [sui::tx_context](https://docs.sui.io/references/framework/sui-framework/tx_context)             | 包含了 `TxContext` 结构体及其相关方法用于读取                       | [交易上下文](./transaction-context.md)         |
| [sui::object](https://docs.sui.io/references/framework/sui-framework/object)                     | 定义了创建对象所需的 `UID` 和 `ID` 类型                              | [它以一个对象开始](./../object)                 |
| [sui::clock](https://docs.sui.io/references/framework/sui-framework/clock)                       | 定义了 `Clock` 类型及其方法                                          | [纪元和时间](./epoch-and-time.md)              |
| [sui::dynamic_field](https://docs.sui.io/references/framework/sui-framework/dynamic_field)       | 实现了添加、使用和移除动态字段的方法                                  | [动态字段](./dynamic-fields.md)                |
| [sui::dynamic_object_field](https://docs.sui.io/references/framework/sui-framework/dynamic_object_field) | 实现了添加、使用和移除动态对象字段的方法                         | [动态对象字段](./dynamic-object-fields.md)     |
| [sui::event](https://docs.sui.io/references/framework/sui-framework/event)                       | 允许为链下监听器发出事件                                            | [事件](./events.md)                           |
| [sui::package](https://docs.sui.io/references/framework/sui-framework/package)                   | 定义了 `Publisher` 类型和包升级方法                                   | [发布者](./publisher.md), [包升级](./package-upgrades.md) |
| [sui::display](https://docs.sui.io/references/framework/sui-framework/display)                   | 实现了 `Display` 对象及其创建和更新方法                               | [显示](./display.md)                          |

</div>

## 集合模块

<div class="modules-table">

| 模块                                                                                         | 描述                                                | 章节                                                   |
| -------------------------------------------------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------ |
| [sui::vec_set](https://docs.sui.io/references/framework/sui-framework/vec_set)                 | 实现了集合类型                                      | [集合](./collections.md)                                |
| [sui::vec_map](https://docs.sui.io/references/framework/sui-framework/vec_map)                 | 使用向量键实现了映射                                  | [集合](./collections.md)                                |
| [sui::table](https://docs.sui.io/references/framework/sui-framework/table)                     | 实现了 `Table` 类型及其与之交互的方法                   | [动态集合](./dynamic-collections.md)                     |
| [sui::linked_table](https://docs.sui.io/references/framework/sui-framework/linked_table)       | 实现了 `LinkedTable` 类型及其与之交互的方法             | [动态集合](./dynamic-collections.md)                     |
| [sui::bag](https://docs.sui.io/references/framework/sui-framework/bag)                         | 实现了 `Bag` 类型及其与之交互的方法                     | [动态集合](./dynamic-collections.md)                     |
| [sui::object_table](https://docs.sui.io/references/framework/sui-framework/object_table)       | 实现了 `ObjectTable` 类型及其与之交互的方法             | [动态集合](./dynamic-collections.md)                     |
| [sui::object_bag](https://docs.sui.io/references/framework/sui-framework/object_bag)           | 实现了 `ObjectBag` 类型及其与之交互的方法               | [动态集合](./dynamic-collections.md)                     |

</div>

## 实用工具模块

<div class="modules-table">

| 模块                                                                                         | 描述                                                        | 章节                                                   |
| -------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------ |
| [sui::bcs](https://docs.sui.io/references/framework/sui-framework/bcs)                         | 实现了 BCS 编码和解码函数                                     | [二进制规范序列化](./bcs.md)                            |

## 导出地址

Sui Framework 导出了两个命名地址：`sui = 0x2` 和 `std = 0x1`，来自于标准库依赖。

```toml
[addresses]
sui = "0x2"

# 从 MoveStdlib 依赖中导出
std = "0x1"
```

## 隐式导入

就像 [标准库](./../move-basics/standard-library.md#implicit-imports) 一样，一些模块和类型在 Sui Framework 中是隐式导入的。以下是可以在没有显式 `use` 导入的情况下使用的模块和类型列表：

- sui::object
- sui::object::ID
- sui::object::UID
- sui::tx_context
- sui::tx_context::TxContext
- sui::transfer

## 源代码

Sui Framework 的源代码可以在 [Sui 仓库](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages/sui-framework/sources) 中找到。

<!--

模块:

货币:
- sui::pay
- sui::sui
- sui::coin
- sui::token
- sui::balance
- sui::deny_list

商业:
- sui::kiosk
- sui::display
- sui::kiosk_extension
- sui::transfer_policy


- sui::bcs
- sui::hex
- sui::math
- sui::types
- sui::borrow


- sui::authenticator

- sui::priority_queue
- sui::table_vec

- sui::url
- sui::versioned

- sui::prover
- sui::random

- sui::bls12381
- sui::ecdsa_k1
- sui::ecdsa_r1
- sui::ecvrf
- sui::ed25519
(还提及验证器 16 增长)
- sui::group_ops
- sui::hash
- sui::hmac
- sui::poseidon
- sui::zklogin_verified_id
- sui::zklogin_verified_issuer

 -->
