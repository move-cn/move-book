# UID和ID

`UID`类型在`sui::object`模块中定义，是围绕`ID`类型包装而成，而`ID`类型则是围绕`address`类型包装。在Sui中，UID保证是唯一的，且在对象被删除后不可再次使用。

```move
// 文件：sui-framework/sources/object.move
/// UID是对象的唯一标识符
public struct UID has store {
    id: ID
}

/// ID是地址的包装器
public struct ID has store, drop {
    bytes: address
}
```

<!-- 用户目前对TxContext还不了解... -->

## 新UID的生成：

- UID是从`tx_hash`和递增的`index`派生而来的。
- `derive_id`函数在`sui::tx_context`模块中实现，因此生成UID需要TxContext。
- Sui验证器不允许使用未在同一函数中创建的UID。这防止了在对象被解包后预先生成和重复使用UID。

使用`object::new(ctx)`函数可以创建新的UID，它接受对TxContext的可变引用，并返回一个新的UID。

```move
let ctx = &mut tx_context::dummy();
let uid = object::new(ctx);
```

在Sui中，`UID`充当对象的表征，并允许定义对象的行为和特征。其中一个关键特性是[动态字段]()，这得益于`UID`类型的显式定义。此外，它还允许进行后文将要解释的[对象转移（TTO）]()。

## UID的生命周期

`UID`类型通过`object::new(ctx)`函数创建，并通过`object::delete(uid)`函数销毁。`object::delete`函数通过值消耗UID，并且除非值是从对象中解包出来的，否则不可能删除它。

```move
let ctx = &mut tx_context::dummy();

let char = Character {
    id: object::new(ctx)
};

let Character { id } = char;
id.delete();
```

## 保持UID

在对象结构解包后，并不需要立即删除`UID`。有时它可能携带[动态字段](./../programmability/dynamic-fields.md)或通过[对象转移](./transfer-to-object.md)传输到它的对象。在这种情况下，UID可以被保留并存储在一个单独的对象中。

## 删除的证明

返回对象的UID的能力可以用于所谓的“删除证明”模式。虽然这是一个不常用的技术，但在某些情况下很有用，例如，创建者或应用程序可以通过交换已删除ID来激励对象的删除。

在框架开发中，这种方法可以用来忽略或绕过对“获取”对象施加的某些限制。例如，如果有一个强制执行传输逻辑的容器，类似于Kiosk，通过提供删除证明就可以特殊情况下跳过检查。

这是一个值得探索和研究的开放主题，可以以各种方式使用。

## ID

谈到`UID`时，我们也应该提到`ID`类型。它是围绕`address`类型的包装器，用于表示地址指针。通常，`ID`用于指向一个对象，但没有限制，也没有保证`ID`指向一个现有对象。

> ID可以作为事务参数在[事务块](./../concepts/what-is-a-transaction.md)中接收。此外，可以使用`to_id()`函数从一个`address`值创建ID。

<!--
```move

TODO: !!!!

```
-->

## fresh_object_address

TxContext提供了`fresh_object_address`函数，可以用于创建唯一的地址和ID。这在一些应用程序中分配唯一标识符给用户行为，例如市场中的order_id，可能非常有用。