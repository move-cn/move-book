# Structs and Resources

一个 _结构体_ 是一个用户定义的数据结构，包含有类型的字段。结构体可以存储任何非引用、非元组类型，包括其他结构体。

结构体可用于定义所有“资产”值或无限制值，其操作可以由结构体的 [能力](./abilities.md) 控制。默认情况下，结构体是线性的和短暂的。这意味着它们不能被复制、不能被丢弃，也不能被存储在存储中。这意味着所有值必须通过所有权转移（线性）来处理，并且在程序执行结束时必须处理这些值（短暂）。我们可以通过赋予结构体 [能力](./abilities.md) 来放宽这种行为，允许值被复制或丢弃，并且可以存储在存储中或定义存储模式。

## 定义结构体

结构体必须在模块内定义，结构体的字段可以是命名的或位置的：

```move
module a::m {
    public struct Foo { x: u64, y: bool }
    public struct Bar {}
    public struct Baz { foo: Foo, }
    //                          ^ 注意：在结尾处加逗号是允许的

    public struct PosFoo(u64, bool)
    public struct PosBar()
    public struct PosBaz(Foo)
}
```

结构体不能是递归的，因此以下定义是无效的：

```move
public struct Foo { x: Foo }
//                     ^ 错误！递归定义

public struct A { b: B }
public struct B { a: A }
//                   ^ 错误！递归定义

public struct D(D)
//              ^ 错误！递归定义
```

### 可见性

正如你可能注意到的，所有结构体都声明为 `public`。这意味着结构体的类型可以从任何其他模块引用。然而，结构体的字段，以及创建或销毁结构体的能力，仍然是在定义结构体的模块内部。

在未来，我们计划添加将结构体声明为 `public(package)` 或作为内部的功能，类似于 [函数](./functions.md#visibility)。

### 能力

如上所述，默认情况下，结构体声明为线性和短暂的。因此，为了允许值在这些方式下使用（例如，复制、丢弃、存储在 [对象](./abilities/object.md) 中，或用于定义可存储的 [对象](./abilities/object.md)），可以通过注释使用 `has <ability>` 来赋予结构体 [能力](./abilities.md)：

```move
module a::m {
    public struct Foo has copy, drop { x: u64, y: bool }
}
```

能力声明可以出现在结构体字段之前或之后。然而，只能使用其中一个，不能同时使用两者。如果在结构体字段之后声明能力，则能力声明必须以分号结尾：

```move
module a::m {
    public struct PreNamedAbilities has copy, drop { x: u64, y: bool }
    public struct PostNamedAbilities { x: u64, y: bool } has copy, drop;
    public struct PostNamedAbilitiesInvalid { x: u64, y: bool } has copy, drop
    //                                                                        ^ 错误！缺少分号

    public struct NamedInvalidAbilities has copy { x: u64, y: bool } has drop;
    //                                                               ^ 错误！重复的能力声明

    public struct PrePositionalAbilities has copy, drop (u64, bool)
    public struct PostPositionalAbilities (u64, bool) has copy, drop;
    public struct PostPositionalAbilitiesInvalid (u64, bool) has copy, drop
    //                                                                     ^ 错误！缺少分号
    public struct InvalidAbilities has copy (u64, bool) has drop;
    //                                                  ^ 错误！重复的能力声明
}
```

更多细节，请参阅关于 [注释结构体和枚举的能力](./abilities.md#annotating-structs-and-enums) 的部分。

### 命名

结构体的名称必须以大写字母 `A` 到 `Z` 开头。在第一个字母之后，结构体名称可以包含下划线 `_`、字母 `a` 到 `z`、字母 `A` 到 `Z` 或数字 `0` 到 `9`。

```move
public struct Foo {}
public struct BAR {}
public struct B_a_z_4_2 {}
public struct P_o_s_Foo()
```

这种以 `A` 到 `Z` 开头的命名限制是为了为未来的语言功能留出空间。它可能会被移除，也可能会在以后保留。

## 使用结构体

### 创建结构体

可以通过指定结构体名称，后跟每个字段的值来创建（或“打包”）结构体类型的值。

对于具有命名字段的结构体，字段的顺序不重要，但必须提供字段名称。对于具有位置字段的结构体，字段的顺序必须与结构体定义中字段的顺序相匹配，并且必须使用 `()` 而不是 `{}` 来括起参数。

```move
module a::m {
    public struct Foo has drop { x: u64, y: bool }
    public struct Baz has drop { foo: Foo }
    public struct Positional(u64, bool) has drop;

    fun example() {
        let foo = Foo { x: 0, y: false };
        let baz = Baz { foo: foo };
        // 注意：位置结构体值是使用括号创建的，基于位置而不是名称。
        let pos = Positional(0, false);
        let pos_invalid = Positional(false, 0);
        //                           ^ 错误！字段顺序不正确且类型不匹配。
    }
}
```

对于具有命名字段的结构体，如果有与字段名称相同的局部变量，可以使用以下简写：

```move
let baz = Baz { foo: foo };
// 等同于
let baz = Baz { foo };
```

这有时被称为“字段名捕捉”。

### 通过模式匹配销毁结构体

可以通过将结构体值绑定或分配到模式中来销毁结构体值，使用的语法与构造它们的语法类似。

```move
module a::m {
    public struct Foo { x: u64, y: bool }
    public struct Bar(Foo)
    public struct Baz {}
    public struct Qux()

    fun example_destroy_foo() {
        let foo = Foo { x: 3, y: false };
        let Foo { x, y: foo_y } = foo;
        //        ^ shorthand for `x: x`

        // two new bindings
        //   x: u64 = 3
        //   foo_y: bool = false
    }

    fun example_destroy_foo_wildcard() {
        let foo = Foo { x: 3, y: false };
        let Foo { x, y: _ } = foo;

        // only one new binding since y was bound to a wildcard
        //   x: u64 = 3
    }

    fun example_destroy_foo_assignment() {
        let x: u64;
        let y: bool;
        Foo { x, y } = Foo { x: 3, y: false };

        // mutating existing variables x and y
        //   x = 3, y = false
    }

    fun example_foo_ref() {
        let foo = Foo { x: 3, y: false };
        let Foo { x, y } = &foo;

        // two new bindings
        //   x: &u64
        //   y: &bool
    }

    fun example_foo_ref_mut() {
        let foo = Foo { x: 3, y: false };
        let Foo { x, y } = &mut foo;

        // two new bindings
        //   x: &mut u64
        //   y: &mut bool
    }

    fun example_destroy_bar() {
        let bar = Bar(Foo { x: 3, y: false });
        let Bar(Foo { x, y }) = bar;
        //            ^ nested pattern

        // two new bindings
        //   x: u64 = 3
        //   y: bool = false
    }

    fun example_destroy_baz() {
        let baz = Baz {};
        let Baz {} = baz;
    }

    fun example_destroy_qux() {
        let qux = Qux();
        let Qux() = qux;
    }
}
```
### 访问结构体字段

结构体的字段可以使用点操作符 `.` 进行访问。

对于具有命名字段的结构体，可以通过字段名称进行访问：

```move
public struct Foo { x: u64, y: bool }
let foo = Foo { x: 3, y: true };
let x = foo.x;  // x == 3
let y = foo.y;  // y == true
```

对于位置结构体，可以通过它们在结构体定义中的位置进行访问：

```move
public struct PosFoo(u64, bool)
let pos_foo = PosFoo(3, true);
let x = pos_foo.0;  // x == 3
let y = pos_foo.1;  // y == true
```

在不借用或复制结构体字段的情况下访问它们受字段能力约束的限制。更多详情请参阅 [借用结构体和字段](#borrowing-structs-and-fields) 和 [读取和写入字段](#reading-and-writing-fields) 部分。

### 借用结构体和字段

可以使用 `&` 和 `&mut` 操作符创建对结构体或字段的引用。这些例子包含了一些可选的类型注释（例如，`: &Foo`）来展示操作的类型。

```move
let foo = Foo { x: 3, y: true };
let foo_ref: &Foo = &foo;
let y: bool = foo_ref.y;         // 通过引用读取结构体的字段
let x_ref: &u64 = &foo.x;        // 通过扩展对结构体的引用借用字段

let x_ref_mut: &mut u64 = &mut foo.x;
*x_ref_mut = 42;            // 通过可变引用修改字段
```

可以借用嵌套结构体的内部字段：

```move
let foo = Foo { x: 3, y: true };
let bar = Bar(foo);

let x_ref = &bar.0.x;
```

你也可以通过对结构体的引用借用字段：

```move
let foo = Foo { x: 3, y: true };
let foo_ref = &foo;
let x_ref = &foo_ref.x;
// 这与 let x_ref = &foo.x 的效果相同
```

### 读取和写入字段

如果需要读取并复制字段的值，可以解引用借用的字段：

```move
let foo = Foo { x: 3, y: true };
let bar = Bar(copy foo);
let x: u64 = *&foo.x;
let y: bool = *&foo.y;
let foo2: Foo = *&bar.0;
```

使用点操作符 `.` 可以读取结构体的字段，而不需要借用。与[解引用](./primitive-types/references.md#reading-and-writing-through-references)一样，字段类型必须具有 `copy` [能力](./abilities.md)。

```move
let foo = Foo { x: 3, y: true };
let x = foo.x;  // x == 3
let y = foo.y;  // y == true
```

点操作符可以链式调用以访问嵌套字段：

```move
let bar = Bar(Foo { x: 3, y: true });
let x = bar.0.x; // x == 3;
```

但是，这不允许包含非原始类型字段的结构体，如向量或其他结构体：

```move
let foo = Foo { x: 3, y: true };
let bar = Bar(foo);
let foo2: Foo = *&bar.0;
let foo3: Foo = bar.0; // 错误! 必须添加显式复制 *&
```

我们可以借用结构体的字段以赋予它新值：

```move
let mut foo = Foo { x: 3, y: true };
*&mut foo.x = 42;     // foo = Foo { x: 42, y: true }
*&mut foo.y = !foo.y; // foo = Foo { x: 42, y: false }
let mut bar = Bar(foo);               // bar = Bar(Foo { x: 42, y: false })
*&mut bar.0.x = 52;                   // bar = Bar(Foo { x: 52, y: false })
*&mut bar.0 = Foo { x: 62, y: true }; // bar = Bar(Foo { x: 62, y: true })
```

与解引用类似，我们可以直接使用点操作符来修改字段。在这两种情况下，字段类型必须具有 `drop` [能力](./abilities.md)。

```move
let mut foo = Foo { x: 3, y: true };
foo.x = 42;     // foo = Foo { x: 42, y: true }
foo.y = !foo.y; // foo = Foo { x: 42, y: false }
let mut bar = Bar(foo);         // bar = Bar(Foo { x: 42, y: false })
bar.0.x = 52;                   // bar = Bar(Foo { x: 52, y: false })
bar.0 = Foo { x: 62, y: true }; // bar = Bar(Foo { x: 62, y: true })
```

点语法用于通过结构体的引用进行赋值也适用：

```move
let foo = Foo { x: 3, y: true };
let foo_ref = &mut foo;
foo_ref.x = foo_ref.x + 1;
```

## 特权结构体操作

大多数针对结构体类型 `T` 的结构体操作只能在声明 `T` 的模块内部执行：

- 结构体类型只能在定义结构体的模块内部创建（"packed"）、销毁（"unpacked"）。
- 结构体的字段只能在定义结构体的模块内部访问。

遵循这些规则，如果你想在模块外修改结构体，需要为其提供公共 API。本章结尾包含一些示例。

然而，如上面的[可见性部分](#visibility)所述，结构体 _类型_ 对其他模块始终可见。

```move
module a::m {
    public struct Foo has drop { x: u64 }

    public fun new_foo(): Foo {
        Foo { x: 42 }
    }
}

module a::n {
    use a::m::Foo;

    public struct Wrapper has drop {
        foo: Foo
        //   ^ 类型是公共的，因此有效
    }

    fun f1(foo: Foo) {
        let x = foo.x;
        //      ^ 错误! 无法在 `a::m` 外部访问 `Foo` 的字段
    }

    fun f2() {
        let foo_wrapper = Wrapper { foo: m::new_foo() };
        //                               ^ 函数是公共的，因此有效
    }
}
```

## 所有权

如上文[定义结构体](#defining-structs)中提到的，默认情况下，结构体是线性和短暂的。这意味着它们不能被复制或丢弃。这一特性在模拟现实世界资产（如货币）时非常有用，因为你不希望货币被复制或在流通中丢失。

```move
module a::m {
    public struct Foo { x: u64 }

    public fun copying() {
        let foo = Foo { x: 100 };
        let foo_copy = copy foo; // 错误! 复制需要 'copy' 能力
        let foo_ref = &foo;
        let another_copy = *foo_ref // 错误! 解引用需要 'copy' 能力
    }

    public fun destroying_1() {
        let foo = Foo { x: 100 };

        // 错误! 当函数返回时，foo 仍包含值。
        // 这种销毁需要 'drop' 能力
    }

    public fun destroying_2(f: &mut Foo) {
        *f = Foo { x: 100 } // 错误!
                            // 通过写入销毁旧值需要 'drop' 能力
    }
}
```

要修复 `fun destroying_1` 示例，需要手动“解包”值：

```move
module a::m {
    public struct Foo { x: u64 }

    public fun destroying_1_fixed() {
        let foo = Foo { x: 100 };
        let Foo { x: _ } = foo;
    }
}
```

请记住，只有在定义结构体的模块中才能解构结构体。这可以用来在系统中强制执行某些不变量，例如货币的保存。

另一方面，如果你的结构体不代表有价值的东西，可以添加 `copy` 和 `drop` 能力，以获得更符合其他编程语言习惯的结构体值：

```move
module a::m {
    public struct Foo has copy, drop { x: u64 }

    public fun run() {
        let foo = Foo { x: 100 };
        let foo_copy = foo;
        //             ^ 这段代码复制了 foo，
        //             而 `let x = move foo` 会移动 foo

        let x = foo.x;            // x = 100
        let x_copy = foo_copy.x;  // x = 100

        // 当函数返回时，foo 和 foo_copy 都被隐式丢弃
    }
}
```

## 存储

结构体可以用来定义存储模式，但具体细节因 Move 的不同部署而异。有关更多详情，请参阅 [`key` 能力](./abilities.md#key) 和 [Sui 对象](./abilities/object.md) 的文档。
