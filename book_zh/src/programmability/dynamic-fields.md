# 动态字段

Sui对象模型允许对象作为_动态字段_附加到其他对象上。这种行为类似于其他编程语言中的`Map`。然而，与Move中的严格类型化`Map`不同（我们在[集合](./collections.md)部分中已介绍），动态字段允许附加任意类型的对象。在前端开发中，类似的方法是JavaScript对象类型，它允许动态存储任何类型的数据。

> 动态字段可以附加到一个对象上的数量没有限制。因此，动态字段可以用于存储大量数据，这些数据不适合对象大小限制。

动态字段允许广泛的应用，从将数据拆分成更小的部分以避免[对象大小限制](./../guides/building-against-limits.md)，到将对象作为应用程序逻辑的一部分附加。

## 定义

动态字段在[Sui框架](./sui-framework.md)的`sui::dynamic_field`模块中定义。它们通过一个_名称_附加到对象的`UID`上，可以使用该名称进行访问。每个对象只能附加一个给定名称的字段。

文件：sui-framework/sources/dynamic_field.move

```move
/// 用于存储字段和值的内部对象
public struct Field<Name: copy + drop + store, Value: store> has key {
    /// 由对象ID、字段名称和类型的哈希值决定，即hash(parent.id || name || Name)
    id: UID,
    /// 该字段名称的值
    name: Name,
    /// 绑定到该字段的值
    value: Value,
}
```

如定义所示，动态字段存储在一个内部`Field`对象中，该对象的`UID`是基于对象ID、字段名称和字段类型的哈希值生成的。`Field`对象包含字段名称和绑定到该字段的值。`Name`和`Value`类型参数上的约束定义了键和值必须具备的能力。

## 用法

动态字段的方法很简单：可以用`add`添加字段，用`remove`删除字段，并用`borrow`和`borrow_mut`读取字段。此外，`exists_`方法可以用来检查字段是否存在（对于更严格的类型检查，有一个`exists_with_type`方法）。

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-fields.move:usage}}
}
```

在上面的例子中，我们定义了一个`Character`对象和两种不同类型的配件，这些配件无法一起放入一个向量中。然而，动态字段允许我们在单个对象中存储它们。两个对象通过`vector<u8>`（字节字符串字面量）附加到`Character`，并可以使用各自的键进行访问。

如你所见，当我们将配件附加到`Character`时，是通过_值_传递的。换句话说，两个值都被移动到一个新的作用域，它们的所有权被转移到`Character`对象。如果我们改变`Character`对象的所有权，配件也会随之移动。

我们应该强调的动态字段的最后一个重要属性是它们_通过其父对象进行访问_。这意味着`Hat`和`Mustache`对象不能直接访问，并遵循与父对象相同的规则。

## 外部类型作为动态字段

动态字段允许对象携带任何类型的数据，包括其他模块中定义的那些。这是由于它们的泛型性质和对类型参数的相对弱约束。让我们通过将一些不同的值附加到一个`Character`对象来说明这一点。

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-fields.move:foreign_types}}
```

在这个例子中，我们展示了如何为动态字段的_名称_和_值_使用不同的类型。`String`通过`vector<u8>`名称附加，`u64`通过`u32`名称附加，`bool`通过`bool`名称附加。使用动态字段可以实现任何可能性！

## 孤立的动态字段

> 为防止孤立的动态字段，请使用[动态集合类型](./dynamic-collections.md)，如`Bag`，它们会跟踪动态字段，并在有附加字段时不允许解包。

用于删除UID的`object::delete()`函数不跟踪动态字段，不能防止动态字段变成孤立字段。一旦父UID被删除，动态字段不会自动删除，它们会变成孤立字段。这意味着动态字段仍然存储在区块链中，但它们将永远无法再次访问。

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-fields.move:orphan_fields}}
```

孤立的对象不属于存储回扣的范畴，存储费用将保持未认领状态。在解包对象时避免孤立动态字段的一种方法是返回`UID`并将其临时存储在某处，直到动态字段被删除并得到适当处理。

## 自定义类型作为字段名称

在上面的例子中，我们使用原始类型作为字段名称，因为它们具有所需的能力。但使用自定义类型作为字段名称时，动态字段变得更加有趣。这允许更结构化地存储数据，并且还允许保护字段名称不被其他模块访问。

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-fields.move:custom_type}}
```

我们在上面定义的两个字段名称是`AccessoryKey`和`MetadataKey`。`AccessoryKey`有一个`String`字段，因此可以使用不同的`name`值多次使用。`MetadataKey`是一个空键，只能附加一次。

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-fields.move:custom_type_usage}}
```

如你所见，自定义类型确实可以作为字段名称，但只要它们可以由模块构造，换句话说 - 如果它们是模块的_内部_并在其中定义。这种对结构打包的限制可以在应用程序设计中开辟新的途径。

这种方法在[对象能力](./object-capability.md)模式中使用，其中应用程序可以授权外部对象在其中执行操作，同时不向其他模块公开能力。

## 暴露UID

<div class="warning">

对`UID`的可变访问是一种安全风险。将你的类型的`UID`作为可变引用暴露可能导致对象的动态字段被意外修改或移除。此外，它影响[转移到对象](./../storage/transfer-to-object.md)和[动态对象字段](./dynamic-object-fields.md)。在暴露`UID`作为可变引用之前，请确保理解其影响。

</div>

由于动态字段附加到`UID`上，它们在其他模块中的使用取决于`UID`是否可以访问。默认情况下，结构可见性保护`id`字段，不允许其他模块直接访问它。然而，如果有一个返回`UID`引用的公共访问器方法，动态字段可以在其他模块中读取。

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-fields.move:exposed_uid}}
```

在上面的例子中，我们展示了如何暴露`Character`对象的`UID`。此解决方案可能适用于某些应用程序，但请记住，暴露的`UID`允许读取附加到对象的_任何_动态字段。

如果你只需要在包内暴露`UID`，请使用限制性可见性，如`public(package)`，或者更好的是 - 使用更具体的访问器方法，只允许读取特定字段。

```move
{{#include ../../../packages/samples/sources/programmability/dynamic-fields.move:exposed_uid_measures}}
```

## 动态字段与常规字段

动态字段比常规字段更昂贵，因为它们需要额外的存储和访问成本。它们的灵活性是有代价的，在决定使用动态字段还是常规字段时，重要的是理解其影响。

## 限制

动态字段不受[对象大小限制](./../guides/building-against-limits.md)的约束，可以用于存储大量数据。然而，它们仍然受[动态字段创建限制](./../guides/building-against-limits.md)的约束，每个事务的字段数量限制为1000个。

## 应用

动态字段在任何复杂度的应用程序中都可以发挥关键作用。它们打开了各种不同的用例，从存储异构数据到将对象作为应用程序逻辑的一部分附加。基于定义它们_稍后_并更改字段类型的能力，它们允许某些[可升级性实践](./../guides/upgradeability-practices.md)。

## 下一步

在下一节中，我们将介绍[动态对象字段](./dynamic-object-fields.md)，并解释它们与动态字段的区别，以及使用它们的影响。