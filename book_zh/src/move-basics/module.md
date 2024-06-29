模块是 Move 中的代码组织基本单元。模块用于组织和隔离代码，模块的所有成员默认情况下对模块私有。在本节中，您将学习如何定义模块，声明其成员以及如何从其他模块访问它们。

## 模块声明

使用 `module` 关键字后跟包地址、模块名称和模块体在花括号 `{}` 内来声明模块。模块名称应采用 `snake_case` 形式，即所有小写字母，单词之间用下划线分隔。模块名称在包内必须是唯一的。

通常，`sources/` 文件夹中的单个文件包含一个模块。文件名应与模块名称匹配 - 例如，`donut_shop` 模块应存储在 `donut_shop.move` 文件中。您可以在[Coding Conventions](../special-topics/coding-conventions.md)部分了解更多有关编码约定的信息。

```move
{{#include ../../../packages/samples/sources/move-basics/module.move:module}}
```

模块的成员包括结构体、函数、常量和导入：

- [结构体](./struct.md)
- [函数](./function.md)
- [常量](./constants.md)
- [导入](./importing-modules.md)
- [结构体方法](./struct-methods.md)

## 地址 / 命名地址

模块地址可以指定为地址_字面量_（不需要 `@` 前缀）或在[包清单](../concepts/manifest.md)中指定的命名地址。在下面的示例中，两者是相同的，因为在 `Move.toml` 的 `[addresses]` 部分有一个 `book = "0x0"` 记录。

```move
{{#include ../../../packages/samples/sources/move-basics/module.move:address_literal}}
```

在 `Move.toml` 中的地址部分：

```toml
# Move.toml
[addresses]
book = "0x0"
```

## 模块成员

模块成员声明在模块体内部。为了说明这一点，让我们定义一个简单的模块，其中包含一个结构体、一个函数和一个常量：

```move
{{#include ../../../packages/samples/sources/move-basics/module.move:members}}
```

## 进一步阅读

- 在 Move 参考文档中阅读有关[模块](/reference/modules.html)的更多信息。