# 标准库

Move 标准库提供了用于本机类型和操作的功能集合。它是一组标准的模块集，不涉及存储操作，而是提供基本工具来处理和操作数据。它是[Sui 框架](../programmability/sui-framework.md)的唯一依赖，并与之一起导入使用。

## 最常见的模块

在本书中，我们详细讨论了标准库中大多数模块，但也有助于概述其功能，让您对可用的功能和实现它们的模块有所了解。

<div class="modules-table">

| 模块                                                                           | 描述                                           | 章节                        |
| ------------------------------------------------------------------------------ | ---------------------------------------------- | --------------------------- |
| [std::string](https://docs.sui.io/references/framework/move-stdlib/string)     | 提供基本的字符串操作                           | [字符串](./string.md)       |
| [std::ascii](https://docs.sui.io/references/framework/move-stdlib/ascii)       | 提供基本的 ASCII 操作                          | [字符串](./string.md)       |
| [std::option](https://docs.sui.io/references/framework/move-stdlib/option)     | 实现 `Option<T>` 类型                          | [Option](./option.md)       |
| [std::vector](https://docs.sui.io/references/framework/move-stdlib/vector)     | 对向量类型进行本地操作                         | [Vector](./vector.md)       |
| [std::bcs](https://docs.sui.io/references/framework/move-stdlib/bcs)           | 包含 `bcs::to_bytes()` 函数                    | [BCS](../move-basics/bcs.md) |
| [std::address](https://docs.sui.io/references/framework/move-stdlib/address)   | 包含单一的 `address::length` 函数              | [地址](./address.md)        |
| [std::type_name](https://docs.sui.io/references/framework/move-stdlib/type_name) | 允许运行时类型反射                             | [类型反射](./type-reflection.md) |
| std::hash                                                                      | 哈希函数：`sha2_256` 和 `sha3_256`              | [加密和哈希](../programmability/cryptography-and-hashing.md) |
| std::debug                                                                     | 包含调试函数，仅在 **测试** 模式下可用           | [调试](./debugging.md)      |
| std::bit_vector                                                                | 提供位向量操作                                 | -                           |
| std::fixed_point32                                                             | 提供 `FixedPoint32` 类型                       | -                           |

</div>

## 导出的地址

标准库导出了一个命名地址 - `std = 0x1`。

```toml
[addresses]
std = "0x1"
```

## 隐式导入

一些模块会隐式导入，可以在模块中直接使用而无需显式导入。对于标准库来说，这些模块和类型包括：

- std::vector
- std::option
- std::option::Option

## 不依赖于 Sui 框架导入 std

Move 标准库可以直接导入到包中。然而，仅导入 `std` 是不足以构建有意义的应用程序，因为它不提供任何存储能力，也不能与链上状态进行交互。

```toml
MoveStdlib = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/move-stdlib", rev = "framework/mainnet" }
```

## 源代码

Move 标准库的源代码可在[Sui 仓库](https://github.com/MystenLabs/sui/tree/main/crates/sui-framework/packages/move-stdlib/sources)中找到。