# 能力：简介

Move 具有独特的类型系统，允许自定义 _类型能力_。
[在前一节](./struct.md)中，我们介绍了`struct`定义及其用法。
但是，`Artist`和`Record`结构体的实例在编译代码时必须被解包。这是没有 _能力_ 的结构体的默认行为。

> 在本书中，你会看到名称为“Ability: <名称>”的章节，其中“<名称>”是能力的名称。这些章节将详细介绍该能力，它如何工作以及如何在 Move 中使用。

## 什么是能力？

能力是一种允许类型具有特定行为的方式。它们是结构体声明的一部分，并定义了结构体实例允许的行为。

## 能力语法

使用`has`关键字后跟能力列表在结构体定义中设置能力。能力之间用逗号分隔。Move 支持四种能力：`copy`、`drop`、`key`和`store`，每种能力用于定义结构体实例的特定行为。

```move
/// 这个结构体具有 `copy` 和 `drop` 能力。
struct VeryAble has copy, drop {
    // field: Type1,
    // field2: Type2,
    // ...
}
```

## 概述

能力的快速概述：

> 除引用外，所有内置类型都具有`copy`、`drop`和`store`能力。引用具有`copy`和`drop`能力。

- `copy` - 允许结构体被 _复制_。详见[Ability: Copy](./copy-ability.md)章节。
- `drop` - 允许结构体被 _丢弃_ 或 _舍弃_。详见[Ability: Drop](./drop-ability.md)章节。
- `key` - 允许结构体用作存储中的 _键_。详见[Ability: Key](./../storage/key-ability.md)章节。
- `store` - 允许结构体 _存储_ 在具有 _key_ 能力的结构体中。详见[Ability: Store](./../storage/store-ability.md)章节。

虽然在这里提到它们很重要，但我们将在后续章节中详细介绍每种能力，并给出如何使用它们的适当上下文。

## 没有能力

没有能力的结构体不能被丢弃、复制或存储在存储中。我们称这种结构体为 _Hot Potato_。这是一个玩笑，但也是记住没有能力的结构体就像一个烫手山芋的好方法——它只能被传递，需要特殊处理。Hot Potato 是 Move 中最强大的模式之一，我们在[Hot Potato](./../programmability/hot-potato.md)章节中详细介绍了它。

## 延伸阅读

- Move参考中的[类型能力](/reference/type-abilities.html)。