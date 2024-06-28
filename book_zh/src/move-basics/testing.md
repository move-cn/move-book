# 结构体方法

Move 编译器支持_接收者语法_，允许在结构体实例上定义可调用的方法。这类似于其他编程语言中的方法语法。这是一种方便的方式，可以在结构体的字段上定义操作。

## 方法语法

如果函数的第一个参数是模块内部的结构体，则可以使用 `.` 运算符调用该函数。如果函数使用另一个模块中的结构体，则默认不会将方法与结构体关联起来。在这种情况下，可以使用标准的函数调用语法来调用该函数。

当导入一个模块时，方法会自动与结构体关联起来。

```move
{{#include ../../../packages/samples/sources/move-basics/struct-methods.move:hero}}
```

## 方法别名

对于定义多个结构体及其方法的模块，可以定义方法别名来避免名称冲突，或为结构体提供更好的方法名。

别名的语法如下：

```move
// 用于本地方法关联
use fun function_path as Type.method_name;

// 公共别名
public use fun function_path as Type.method_name;
```

> 公共别名只允许用于同一模块中定义的结构体。如果结构体在另一个模块中定义，仍然可以创建别名，但不能公开。

在下面的示例中，我们更改了 `hero` 模块，并添加了另一种类型 - `Villain`。`Hero` 和 `Villain` 都具有类似的字段名称和方法。为了避免名称冲突，我们为这些方法添加了前缀 `hero_` 和 `villain_`。但是，我们可以为这些方法创建别名，以便在结构体实例上调用时不需要前缀。

```move
{{#include ../../../packages/samples/sources/move-basics/struct-methods.move:hero_and_villain}}
```

正如你所看到的，在测试函数中，我们在 `Hero` 和 `Villain` 的实例上调用了 `health` 方法，而不使用前缀。编译器将自动将方法与结构体关联起来。

## 别名一个外部模块的方法

还可以将在另一个模块中定义的函数与当前模块的结构体关联起来。按照相同的方法，我们可以为在另一个模块中定义的方法创建别名。让我们使用[标准库](./standard-library.md)中的 `bcs::to_bytes` 方法，并将其与 `Hero` 结构体关联起来。这将允许将 `Hero` 结构体序列化为字节向量。

```move
{{#include ../../../packages/samples/sources/move-basics/struct-methods.move:hero_to_bytes}}
```

## 进一步阅读

- 在 Move 参考中的 [方法语法](/reference/method-syntax.html)。