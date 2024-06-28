## 导入模块

Move 通过允许模块导入来实现高模块化和代码重用。同一个包中的模块可以互相导入，新的包也可以依赖现有的包并使用它们的模块。本节将介绍导入模块的基础知识以及如何在您自己的代码中使用它们。

## 导入模块

在同一个包中定义的模块可以互相导入。`use` 关键字后面跟着模块路径，该路径由包地址（或别名）和模块名称组成，两者之间用 `::` 分隔。

```move
// 文件: sources/module_one.move
{{#include ../../../packages/samples/sources/move-basics/importing-modules.move:module_one}}
```

同一个包中定义的另一个模块可以使用 `use` 关键字导入第一个模块。

```move
// 文件: sources/module_two.move
{{#include ../../../packages/samples/sources/move-basics/importing-modules.move:module_two}}
```

## 导入成员

您还可以从模块中导入特定成员。这在您只需要来自模块的单个函数或单个类型时非常有用。语法与导入模块相同，但您在模块路径之后添加成员名称。

```move
{{#include ../../../packages/samples/sources/move-basics/importing-modules.move:members}}
```

## 分组导入

可以使用花括号 `{}` 将导入分组到单个 `use` 语句中。当您需要从同一个模块导入多个成员时，这非常有用。Move 允许对来自同一个模块和来自同一个包的导入进行分组。

```move
{{#include ../../../packages/samples/sources/move-basics/importing-modules.move:grouped}}
```

在 Move 中，单个函数的导入并不常见，因为函数名称可能会重叠并引起混淆。推荐的做法是导入整个模块并使用模块路径来访问函数。类型具有唯一名称，应该单独导入。

要将成员和模块本身一起导入到组导入中，可以使用 `Self` 关键字。`Self` 关键字指的是模块本身，可用于导入模块及其成员。

```move
{{#include ../../../packages/samples/sources/move-basics/importing-modules.move:self}}
```

## 解决命名冲突

从不同模块导入多个成员时，可能会出现命名冲突。例如，如果您导入了两个都具有相同名称的函数的模块，则需要使用模块路径来访问该函数。不同的包中也可能存在具有相同名称的模块。为了解决冲突并避免歧义，Move 提供了 `as` 关键字来重命名导入的成员。

```move
{{#include ../../../packages/samples/sources/move-basics/importing-modules.move:conflict}}
```

## 添加外部依赖项

通过 `sui` 二进制程序生成的新包都包含一个 `Move.toml` 文件，其中对 _Sui 框架_ 包有一个依赖项。Sui 框架依赖于 _标准库_ 包。这两个包都可以在默认配置中使用。包依赖项在 [包清单](./../concepts/manifest.md) 中定义如下：

```toml
[dependencies]
Sui = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }
Local = { local = "../my_other_package" }
```

`dependencies` 部分包含了一个包依赖项列表。键是包的名称，值是 git 导入表或本地路径。git 导入表包含包的 URL、包所在的子目录以及包的修订版本。本地路径是包目录的相对路径。

如果在 `Move.toml` 文件中添加了依赖项，编译器在构建包时会自动获取（稍后会重新获取）依赖项。

## 从另一个包导入模块

通常，包会在 `[addresses]` 部分定义它们的地址，因此您可以使用别名代替地址。例如，您可以使用 `sui::coin` 模块代替 `0x2::coin` 模块。`sui` 别名在 Sui 框架包中定义。类似地，`std` 别名在标准库包中定义，可用于访问标准库模块。

要从另一个包导入模块，请使用 `use` 关键字后跟模块路径。模块路径由包地址（或别名）和模块名称组成，两者之间用 `::` 分隔。

```move
{{#include ../../../packages/samples/sources/move-basics/importing-modules.move:external}}
```