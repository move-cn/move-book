函数是 Move 程序的基本构建块。它们可以从[用户交互](../concepts/user-interaction.md)中调用，也可以从其他函数中调用，并将可执行的代码组织成可重用的单元。函数可以接受参数并返回值。在模块级别，它们使用 `fun` 关键字声明。默认情况下，它们是私有的，只能在模块内部访问。

```move
{{#include ../../../packages/samples/sources/move-basics/function.move:math}}
```

在这个示例中，我们定义了一个名为 `add` 的函数，它接受两个类型为 `u64` 的参数，并返回它们的和。该函数被从同一模块中的 `test_add` 函数调用，后者是一个测试函数。在测试中，我们将 `add` 函数的结果与期望值进行比较，如果结果不同，则终止执行。

## 函数声明

> 在 Move 中，有一个约定，即使用 `snake_case` 命名函数。这意味着函数名称应全部小写，并用下划线分隔单词。例如，`do_something`、`add`、`get_balance`、`is_authorized` 等等。

函数使用 `fun` 关键字声明，后跟函数名称（有效的 Move 标识符），括号中是参数列表，以及返回类型。函数体是一段包含语句和表达式的代码块。函数体中的最后一个表达式是函数的返回值。

```move
{{#include ../../../packages/samples/sources/move-basics/function.move:return_nothing}}
```

## 访问函数

与任何其他模块成员一样，函数可以通过路径导入和访问。路径由模块路径和函数名称组成，用 `::` 分隔。例如，如果在 `book` 包的 `math` 模块中有一个名为 `add` 的函数，则其路径为 `book::math::add`；如果模块已导入，则为 `math::add`。

```move
{{#include ../../../packages/samples/sources/move-basics/function.move:use_math}}
```

## 多返回值

Move 函数可以返回多个值，这在需要从函数返回多个值时非常有用。函数的返回类型是类型的元组。返回值是表达式的元组。

```move
{{#include ../../../packages/samples/sources/move-basics/function.move:tuple_return}}
```

具有元组返回的函数调用的结果必须通过 `let (tuple)` 语法解包到变量中：

```move
{{#include ../../../packages/samples/sources/move-basics/function.move:tuple_return_imm}}
```

如果声明的某些值需要声明为可变的，则在变量名之前放置 `mut` 关键字：

```move
{{#include ../../../packages/samples/sources/move-basics/function.move:tuple_return_mut}}
```

如果某些参数未被使用，则可以用 `_` 符号忽略它们：

```move
{{#include ../../../packages/samples/sources/move-basics/function.move:tuple_return_ignore}}
```

## 进一步阅读

- [Functions](/reference/functions.html) 在 Move 参考文档中