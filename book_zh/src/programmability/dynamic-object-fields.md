# 动态对象字段

> 本节扩展了[动态字段](./dynamic-fields.md)。请先阅读它，以了解动态字段的基本知识。

动态字段的另一种变体是_动态对象字段_，它与常规动态字段有一些不同之处。在本节中，我们将介绍动态对象字段的具体细节，并解释它们与常规动态字段的区别。

> 一般建议是尽量避免使用动态对象字段，除非确实需要通过ID进行直接发现。动态对象字段的额外成本可能无法被其提供的好处所证明。

## 定义

动态对象字段在[Sui框架](./sui-framework.md)的`sui::dynamic_object_fields`模块中定义。它们在许多方面与动态字段相似，但与动态字段不同，动态对象字段对`Value`类型有额外的约束。`Value`必须具有`key`和`store`的组合，而不仅仅是动态字段中的`store`。

它们在框架定义中不那么明确，因为这个概念本身更为抽象：

文件：sui-framework/sources/dynamic_object_fields.move

```move
/// 用于存储字段和值的内部对象
public struct Wrapper<Name> has copy, drop, store {
    name: Name,
}
```

与[动态字段](./dynamic-fields.md#definition)部分中的`Field`类型不同，`Wrapper`类型仅存储字段的名称。值是对象本身，_未被包装_。

`Value`类型的约束在动态对象字段可用的方法中变得明显。以下是`add`函数的签名：

```move
/// 将动态对象字段添加到对象`object: &mut UID`上的由`name: Name`指定的字段中。
/// 如果对象已经具有该名称的字段，则中止并返回`EFieldAlreadyExists`。
public fun add<Name: copy + drop + store, Value: key + store>(
    // 我们在多个地方使用&mut UID进行访问控制
    object: &mut UID,
    name: Name,
    value: Value,
) { /* 实现省略 */ }
```

其余与[动态字段](./dynamic-fields.md#usage)部分中相同的方法对`Value`类型有相同的约束。我们列出它们以供参考：

- `add` - 向对象添加动态对象字段
- `remove` - 从对象中删除动态对象字段
- `borrow` - 从对象中借用动态对象字段
- `borrow_mut` - 从对象中借用动态对象字段的可变引用
- `exists_` - 检查动态对象字段是否存在
- `exists_with_type` - 检查特定类型的动态对象字段是否存在

此外，还有一个`id`方法，它返回`Value`对象的`ID`，而不指定其类型。

## 用法及与动态字段的区别

动态字段和动态对象字段之间的主要区别在于后者只允许将_对象_作为值进行存储。这意味着你不能存储`u64`或`bool`等原始类型。尽管如此，动态对象字段并未被包装成一个单独的对象，这种约束可以被看作一种限制。

> 放宽包装的要求使对象可通过其ID进行链外发现。然而，如果实现了包装对象索引，这一特性可能不再出色，从而使动态对象字段成为冗余特性。

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-object-fields.move:usage}}
```

## 定价差异

动态对象字段比动态字段稍微昂贵一些。由于其内部结构，它们需要两个对象：名称的包装器和值。因此，添加和访问对象字段的成本（加载2个对象相比于动态字段的1个对象）更高。

## 下一步

动态字段和动态对象字段都是强大的功能，允许在应用程序中实现创新的解决方案。然而，它们相对低级，需要仔细处理以避免孤立字段。在下一节中，我们将介绍一个更高级别的抽象 - [动态集合](./dynamic-collections.md)，它可以更有效地管理动态字段和对象。