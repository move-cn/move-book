# 地址类型

<!--

Chapter: Basic Syntax
Goal: Introduce the address type
Notes:
    - a special type
    - named addresses via the Move.toml
    - address literals
    - 0x2 is 0x0000000...02

Links:
    - address concept
    - transaction context
    - Move.toml
    - your first move

 -->

在Move中，为了表示[地址](./../concepts/address.md)，使用了一种特殊的类型称为`address`。它是一个32字节的值，用于表示区块链上的任何地址。地址有两种语法形式：以`0x`为前缀的十六进制地址和命名地址。

```move
{{#include ../../../packages/samples/sources/move-basics/address.move:address_literal}}
```

地址字面量以`@`符号开头，后面跟着一个十六进制数字或标识符。十六进制数字被解释为一个32字节的值。编译器将在[Move.toml](./../concepts/manifest.md)文件中查找该标识符，并将其替换为相应的地址。如果在Move.toml文件中找不到该标识符，编译器将抛出错误。

## 转换

Sui框架提供了一组辅助函数来处理地址。由于地址类型是一个32字节的值，可以将其转换为`u256`类型，反之亦然。它还可以转换为`vector<u8>`类型和从`vector<u8>`类型转换回地址类型。

示例：将地址转换为`u256`类型，然后再转换回来。

```move
{{#include ../../../packages/samples/sources/move-basics/address.move:to_u256}}
```

示例：将地址转换为`vector<u8>`类型，然后再转换回来。

```move
{{#include ../../../packages/samples/sources/move-basics/address.move:to_bytes}}
```

示例：将地址转换为字符串。

```move
{{#include ../../../packages/samples/sources/move-basics/address.move:to_string}}
```

## 进一步阅读

- 在Move参考中的[Address](/reference/primitive-types/address.html)。