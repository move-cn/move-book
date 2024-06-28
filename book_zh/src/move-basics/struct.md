# 使用结构体定义自定义类型

在 Move 的类型系统中，定义自定义类型可以根据应用程序的特定需求进行定制。这不仅仅局限于数据层面，还包括其行为。本节介绍了结构体的定义及其使用方法。

## 结构体

要定义自定义类型，可以使用 `struct` 关键字后跟类型的名称。在名称之后，可以定义结构体的字段。每个字段使用 `field_name: field_type` 语法进行定义，字段定义之间用逗号分隔。字段可以是任何类型，包括其他结构体。

> Move 不支持递归结构体，这意味着结构体不能包含自身作为字段。

```move
{{#include ../../../packages/samples/sources/move-basics/struct.move:def}}
```

在上面的示例中，我们定义了一个 `Record` 结构体，它包含五个字段：`title` 字段是 `String` 类型，`artist` 字段是 `Artist` 类型，`year` 字段是 `u16` 类型，`is_debut` 字段是 `bool` 类型，`edition` 字段是 `Option<u16>` 类型。`edition` 字段的类型是 `Option<u16>`，表示版本号是可选的。

结构体默认是私有的，意味着它们不能被导入和在定义之外的模块中使用。它们的字段也是私有的，无法从模块外部访问。有关不同可见性修饰符的更多信息，请参阅[可见性](./visibility.md)章节。

> 结构体的字段是私有的，只能由定义结构体的模块访问。要在其他模块中读取和写入结构体的字段，必须由定义结构体的模块提供公共函数来访问字段。

## 创建和使用实例

我们已经描述了结构体的定义方式。现在让我们看看如何初始化结构体并使用它。结构体可以使用 `struct_name { field1: value1, field2: value2, ... }` 的语法进行初始化。字段的初始化顺序可以任意，但必须设置所有字段。

```move
{{#include ../../../packages/samples/sources/move-basics/struct.move:pack}}
```

在上面的示例中，我们创建了 `Artist` 结构体的一个实例，并将 `name` 字段设置为字符串 "The Beatles"。

要访问结构体的字段，可以使用 `.` 运算符后跟字段名。

```move
{{#include ../../../packages/samples/sources/move-basics/struct.move:access}}
```

只有定义结构体的模块才能访问其字段（可变和不可变）。因此，上述代码应该与 `Artist` 结构体位于同一个模块中。

## 解构结构体

结构体默认是非可丢弃的，这意味着初始化的结构体值必须被使用：要么存储，要么进行 _解构_。解构结构体意味着将其拆解为其各个字段。可以使用 `let` 关键字后跟结构体名称和字段名称来完成解构。

```move
{{#include ../../../packages/samples/sources/move-basics/struct.move:unpack}}
```

在上面的示例中，我们解构 `Artist` 结构体，并创建了一个新变量 `name`，其值为 `name` 字段的值。因为变量未被使用，编译器会发出警告。为了消除警告，可以使用下划线 `_` 表示变量是有意未使用的。

```move
{{#include ../../../packages/samples/sources/move-basics/struct.move:unpack_ignore}}
```

## 进一步阅读

- 在 Move 参考中查看 [结构体](/reference/structs.html)。