## 集合

集合类型是任何编程语言的基础部分。它们用于存储数据集合，例如项目列表。`vector` 类型已经在 [向量部分](./../move-basics/vector.md) 中介绍过，本章我们将介绍  [Sui 框架](./sui-framework.md) 提供的基于 vect 的集合类型。

## 向量

虽然我们之前已经在 [向量部分](./../move-basics/vector.md) 中介绍过 `vector` 类型，但在新的上下文中再次回顾它还是很有意义的。这一次，我们将介绍如何在对象中使用 `vector` 类型以及如何在应用程序中使用它。

```move
{{#include ../../../packages/samples/sources/programmability/collections.move:vector}}
```

## VecSet

`VecSet` 是一种用于存储一组唯一项目的集合类型。它类似于 `vector`，但它不允许重复的项目。这使得它非常适合存储一组唯一的项目，例如唯一 ID 或地址列表。

尝试插入集合中已经存在的项目，`VecSet` 会报错。

## VecMap

`VecMap` 是一种用于存储键值对映射的集合类型。它类似于 `VecSet`，但它允许您将一个值与集合中的每个项目相关联。这使得它非常适合存储键值对集合，例如地址及其余额列表，或者用户 ID 及其关联数据列表。

`VecMap` 中的键是唯一的，每个键只能与单个值相关联。如果您尝试插入一个键值对，其中键已经存在于映射中，则旧值将被新值替换。

```move
{{#include ../../../packages/samples/sources/programmability/collections.move:vec_map}}
```

## 限制

标准集合类型是一种存储类型数据并保证安全性和一致性的好方法。但是，它们受到可以存储的数据类型的限制（类型系统不允许您在集合中存储错误的类型）；并且它们受到大小的限制（对象大小限制）。它们适用于相对较小的集合和列表，但对于更大的集合，您可能需要使用不同的方法。

集合类型的另一个限制是无法比较它们。因为插入顺序不是确定的，所以尝试将一个 `VecSet` 与另一个 `VecSet` 进行比较可能不会产生预期的结果。

> 此行为会由 linter 捕获并发出警告：_比较类型为 'sui::vec_set::VecSet' 的集合可能会产生意外的结果_

```move
{{#include ../../../packages/samples/sources/programmability/collections.move:vec_set_comparison}}
```

在上面的例子中，比较会失败，因为插入顺序不是确定的，并且两个 `VecSet` 实例可能具有不同的元素顺序。即使两个 `VecSet` 实例包含相同的元素，比较也会失败。

## 总结

* 向量是一种原生类型，允许存储项目列表。
* VecSet 基于向量构建，允许存储唯一项目的集合。
* VecMap 用于以类似映射的结构存储键值对。
* 基于 vect 的集合严格类型化，并受对象大小限制，最适合小型集合和列表。

## 后续

下一节我们将介绍 [动态字段](./dynamic-fields.md) - 这是一种重要的原语，允许使用 [动态集合](./dynamic-collections.md) - 以更灵活但更昂贵的方式存储大型数据集合。
