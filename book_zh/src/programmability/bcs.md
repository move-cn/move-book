# 二进制规范序列化（BCS）

二进制规范序列化（BCS）是一种用于结构化数据的二进制编码格式。最初在Diem中设计，现在已成为Move语言的标准序列化格式。BCS简单、高效、确定性强，并且容易在任何编程语言中实现。

> 完整的格式规范可以在[BCS仓库](https://github.com/zefchain/bcs)中找到。

## 格式

BCS是一种支持无符号整数（最高256位）、选项、布尔值、单位（空值）、定长和变长序列以及映射的二进制格式。该格式设计为确定性，即相同的数据总是会序列化为相同的字节。

> “BCS不是自描述格式。因此，要反序列化消息，必须预先知道消息类型和布局。”——来自[README](https://github.com/zefchain/bcs)

整数以小端格式存储，变长整数使用变长编码方案。序列前缀为其长度（ULEB128），枚举存储为变体的索引加数据，映射存储为有序的键值对序列。

结构体被视为字段的序列，字段按定义顺序序列化，使用与顶层数据相同的规则。

## 使用BCS

[Sui框架](./sui-framework.md)包含`sui::bcs`模块，用于编码和解码数据。编码函数是虚拟机的原生函数，解码函数在Move中实现。

## 编码

要编码数据，可以使用`bcs::to_bytes`函数，将数据引用转换为字节向量。该函数支持编码任何类型，包括结构体。

```move
// 文件: move-stdlib/sources/bcs.move
public native fun to_bytes<T>(t: &T): vector<u8>;
```

以下示例展示了如何使用BCS编码结构体。`to_bytes`函数可以接收任何值并将其编码为字节向量。

```move
{{#include ../../../packages/samples/sources/programmability/bcs.move:encode}}
```

### 编码结构体

结构体的编码类似于简单类型。以下是如何使用BCS编码结构体：

```move
{{#include ../../../packages/samples/sources/programmability/bcs.move:encode_struct}}
```

## 解码

由于BCS不是自描述的且Move是静态类型的，解码需要预先了解数据类型。`sui::bcs`模块提供了各种函数来帮助这个过程。

### 包装API

BCS在Move中实现为一个包装器。解码器按值接收字节，然后调用不同的解码函数（以`peel_*`为前缀）来“剥离”数据。数据从字节中分离，剩余字节保存在包装器中，直到调用`into_remainder_bytes`函数。

```move
{{#include ../../../packages/samples/sources/programmability/bcs.move:decode}}
```

在解码过程中，通常在单个`let`语句中使用多个变量。这使代码更具可读性，并有助于避免不必要的数据复制。

```move
{{#include ../../../packages/samples/sources/programmability/bcs.move:chain_decode}}
```

### 解码向量

虽然大多数基本类型有专门的解码函数，但向量需要特殊处理，具体取决于元素类型。对于向量，首先需要解码向量的长度，然后在循环中解码每个元素。

```move
{{#include ../../../packages/samples/sources/programmability/bcs.move:decode_vector}}
```

对于最常见的场景，`bcs`模块提供了一组基本的函数来解码向量：

- `peel_vec_address(): vector<address>`
- `peel_vec_bool(): vector<bool>`
- `peel_vec_u8(): vector<u8>`
- `peel_vec_u64(): vector<u64>`
- `peel_vec_u128(): vector<u128>`
- `peel_vec_vec_u8(): vector<vector<u8>>` - 字节向量的向量

### 解码选项

[Option](./../move-basics/option.md) 表示为一个0或1个元素的向量。要读取一个选项，可以将其视为向量并检查其长度（第一个字节 - 1或0）。

```move
{{#include ../../../packages/samples/sources/programmability/bcs.move:decode_option}}
```

> 如果需要解码自定义类型的选项，请使用上面的代码片段中的方法。

对于最常见的场景，`bcs`模块提供了一组基本的函数来解码选项：

- `peel_option_address(): Option<address>`
- `peel_option_bool(): Option<bool>`
- `peel_option_u8(): Option<u8>`
- `peel_option_u64(): Option<u64>`
- `peel_option_u128(): Option<u128>`

### 解码结构体

结构体按字段逐个解码，没有标准函数可以自动将字节解码为Move结构体，因为这会违反Move的类型系统。相反，需要手动解码每个字段。

```move
{{#include ../../../packages/samples/sources/programmability/bcs.move:decode_struct}}
```

## 总结

二进制规范序列化是一种高效的结构化数据二进制格式，确保跨平台的一致序列化。Sui框架提供了全面的BCS工具，通过内置函数实现了广泛的功能。