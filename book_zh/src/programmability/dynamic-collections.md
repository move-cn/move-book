# 动态集合

[Sui 框架](./sui-framework.md)提供了多种集合类型，基于[动态字段](./dynamic-fields.md)和[动态对象字段](./dynamic-object-fields.md)的概念构建。这些集合类型旨在以更安全和更易理解的方式存储和管理动态字段和对象。

对于每种集合类型，我们将指定它们使用的基本类型和它们提供的特定功能。

> 与操作 UID 的动态（对象）字段不同，集合类型具有自己的类型，并允许调用[关联函数](./../move-basics/struct-methods.md)。

## 公共概念

所有集合类型共享相同的一组方法，包括：

- `add` - 将字段添加到集合中
- `remove` - 从集合中移除字段
- `borrow` - 从集合中借用字段
- `borrow_mut` - 从集合中借用可变引用字段
- `contains` - 检查集合中是否存在字段
- `length` - 返回集合中字段的数量
- `is_empty` - 检查`length`是否为0

所有集合类型都支持对`borrow`和`borrow_mut`方法使用索引语法。如果在示例中看到方括号，它们将被转换为对`borrow`和`borrow_mut`的调用。

```move
let hat: &Hat = &bag[b"key"];
let hat_mut: &mut Hat = &mut bag[b"key"];

// 等同于
let hat: &Hat = bag.borrow(b"key");
let hat_mut: &mut Hat = bag.borrow_mut(b"key");
```

在示例中，我们不会专注于这些函数，而是关注集合类型之间的区别。

## Bag

正如其名，Bag 表示一组异构值的“袋子”。它是一个简单的非泛型类型，可以存储任何数据。Bag 永远不会允许存在孤立的字段，因为它会跟踪字段的数量，如果不是空的，则不能销毁它。

```move
// 文件：sui-framework/sources/bag.move
public struct Bag has key, store {
    /// 此 Bag 的 ID
    id: UID,
    /// Bag 中键值对的数量
    size: u64,
}
```

由于 Bag 存储任何类型，它提供了额外的方法：

- `contains_with_type` - 检查是否存在特定类型的字段

作为结构字段使用：

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-collections.move:bag_struct}}
```

使用 Bag：

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-collections.move:bag_usage}}
```

## ObjectBag

在`sui::object_bag`模块中定义。与 [Bag](#bag) 相同，但在内部使用[动态对象字段](./dynamic-object-fields.md)。只能存储对象作为值。

## Table

Table 是一个具有固定键和值类型的类型化动态集合。它在`sui::table`模块中定义。

```move
// 文件：sui-framework/sources/table.move
public struct Table<phantom K: copy + drop + store, phantom V: store> has key, store {
    /// 此 Table 的 ID
    id: UID,
    /// Table 中键值对的数量
    size: u64,
}
```

作为结构字段使用：

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-collections.move:table_struct}}
```

使用 Table：

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-collections.move:table_usage}}
```

## ObjectTable

在`sui::object_table`模块中定义。与 [Table](#table) 相同，但在内部使用[动态对象字段](./dynamic-object-fields.md)。只能存储对象作为值。

## 概要

- [Bag](#bag) - 一个简单的集合，可以存储任何类型的数据
- [ObjectBag](#objectbag) - 一个只能存储对象的集合
- [Table](#table) - 一个具有固定键和值类型的类型化动态集合
- [ObjectTable](#objecttable) - 与 Table 相同，但只能存储对象
<!-- [Linked Table](#linkedtable) -->

## LinkedTable

此部分即将推出！

<!-- TODO! -->

<!-- ## Choosing a Collection Type

Depending on the needs of your project, you may choose to -->

<!-- ## LinkedTable

TODO: ... -->