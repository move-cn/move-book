# 向量

向量是在 Move 中存储元素集合的一种本地方式。它们类似于其他编程语言中的数组，但有一些不同之处。在本节中，我们介绍 `vector` 类型及其操作。

## 向量语法

`vector` 类型使用 `vector` 关键字定义，后跟尖括号中的元素类型。元素的类型可以是任何有效的 Move 类型，包括其他向量。Move 提供了一种向量字面量语法，允许你使用 `vector` 关键字后跟方括号来创建向量，其中包含元素（或者对于空向量不包含任何元素）。

```move
{{#include ../../../packages/samples/sources/move-basics/vector.move:literal}}
```

`vector` 类型是 Move 中的内置类型，不需要从模块中导入。然而，向量操作定义在 `std::vector` 模块中，你需要导入该模块才能使用这些操作。

## 向量操作

标准库提供了许多操作向量的方法。以下是一些常用的操作：

- `push_back`: 在向量末尾添加一个元素。
- `pop_back`: 移除向量的最后一个元素。
- `length`: 返回向量中元素的数量。
- `is_empty`: 如果向量为空则返回 true。
- `remove`: 移除给定索引处的元素。

```move
{{#include ../../../packages/samples/sources/move-basics/vector.move:methods}}
```

## 销毁非可丢弃类型的向量

非可丢弃类型的向量不能被丢弃。如果你定义了一个没有 `drop` 能力的类型的向量，则该向量的值不能被忽略。然而，如果向量是空的，编译器要求显式调用 `destroy_empty` 函数。

```move
{{#include ../../../packages/samples/sources/move-basics/vector.move:no_drop}}
```

## 进一步阅读

- 在 Move 参考文档中查看 [向量](/reference/primitive-types/vector.html)。