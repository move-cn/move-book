# Option（选项）

Option 是一种表示可选值的类型，它可能存在，也可能不存在。Move 中的 Option 概念借鉴自 Rust，它是 Move 中非常有用的原语。`Option` 在[标准库](./standard-library.md)中定义，如下所示：

文件：move-stdlib/source/option.move

```move
// 文件：move-stdlib/source/option.move
/// 可能存在，也可能不存在的值的抽象。
struct Option<Element> has copy, drop, store {
    vec: vector<Element>
}
```

> 'std::option' 模块在每个模块中都被隐式导入，您不需要添加导入语句。

`Option` 是一个泛型类型，它接受类型参数 `Element`。它有一个名为 `vec` 的字段，类型为 `vector`，存储着 `Element` 的值。Vector 的长度可以为 0 或 1，用于表示值的存在或不存在。

Option 类型有两个变体：`Some` 和 `None`。`Some` 变体包含一个值，而 `None` 变体表示值的不存在。`Option` 类型用于以类型安全的方式表示值的缺失，并避免使用空或`undefined`值。

## 实际应用

为了展示为什么 Option 类型是必要的，让我们看一个例子。考虑一个应用程序，它接受用户输入并将其存储在一个变量中。有些字段是必需的，而有些是可选的。例如，用户的中间名是可选的。虽然我们可以使用空字符串表示中间名的缺失，但这将需要额外的检查来区分空字符串和缺失的中间名。相反，我们可以使用 `Option` 类型来表示中间名。

```move
{{#include ../../../packages/samples/sources/move-basics/option.move:registry}}
```

在上面的示例中，`middle_name` 字段的类型是 `Option<String>`。这意味着 `middle_name` 字段可以包含一个 `String` 值，也可以为空。这样清晰地表示了中间名是可选的，并避免了额外的检查来区分空字符串和缺失的中间名。

## 使用 Option

要使用 `Option` 类型，您需要导入 `std::option` 模块并使用 `Option` 类型。然后，您可以使用 `some` 或 `none` 方法创建一个 `Option` 值。

```move
{{#include ../../../packages/samples/sources/move-basics/option.move:usage}}
```