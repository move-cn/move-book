# Methods(方法)

为了简化语法，Move中的一些函数可以作为值的“方法”来调用。这是通过使用 `.` 操作符来调用函数实现的，其中 `.` 左侧的值是函数的第一个参数（有时称为接收者）。该值的类型静态决定了调用哪个函数。这与其他一些语言的重要区别在于，这种语法表示的不是动态调用，函数调用在运行时确定。在Move中，所有函数调用都是静态确定的。

简而言之，这种语法的存在使得在调用函数时不必创建 `use` 别名，并且不必显式借用函数的第一个参数。此外，这可以使代码更具可读性，因为它减少了调用函数所需的样板代码，并使得链式调用函数更容易。

## 语法

调用方法的语法如下：

```text
<expression> . <identifier> <[type_arguments],*> ( <arguments> )
```

例如：

```move
coin.value();
*nums.borrow_mut(i) = 5;
```

## 方法解析

当调用方法时，编译器将静态确定调用哪个函数，基于接收者的类型（`.` 左侧的参数）。编译器维护一个从类型和方法名称到模块和函数名称的映射。这个映射是根据当前范围内的 `use fun` 别名以及接收者类型的定义模块中的适当函数创建的。在所有情况下，接收者类型是函数的第一个参数，无论是按值传递还是按引用传递。

在本节中，当我们说一个方法“解析”为一个函数时，我们的意思是编译器将静态地用正常的[函数](./functions.md)调用替换该方法。例如，如果我们有 `x.foo(e)`，其中 `foo` 解析为 `a::m::foo`，则编译器将 `x.foo(e)` 替换为 `a::m::foo(x, e)`，可能会[自动借用](#automatic-borrowing) `x`。

### 定义模块中的函数

在类型的定义模块中，编译器将自动为其类型的任何函数声明创建一个方法别名，当类型是函数的第一个参数时。例如：

```move
module a::m {
    public struct X() has copy, drop, store;
    public fun foo(x: &X) { ... }
    public fun bar(flag: bool, x: &X) { ... }
}
```

函数 `foo` 可以作为类型 `X` 的值的方法调用。然而，`bar` 不能（因为第一个参数不是 `X`，并且不会为 `bool` 创建一个别名，因为 `bool` 不是在该模块中定义的）。例如：

```move
fun example(x: a::m::X) {
    x.foo(); // 有效
    // x.bar(true); 错误!
}
```

### `use fun` 别名

与传统的 [`use`](uses.md) 语句类似，`use fun` 语句创建一个别名，在其当前范围内是局部的。这可以是当前模块或当前表达式块。然而，该别名是关联到一个类型的。

`use fun` 语句的语法如下：

```move
use fun <function> as <type>.<method alias>;
```

这为 `<function>` 创建了一个别名，<type> 可以作为 `<method alias>` 接收。

例如：

```move
module a::cup {
    public struct Cup<T>(T) has copy, drop, store;

    public fun cup_borrow<T>(c: &Cup<T>): &T {
        &c.0
    }

    public fun cup_value<T>(c: Cup<T>): T {
        let Cup(t) = c;
        t
    }

    public fun cup_swap<T: drop>(c: &mut Cup<T>, t: T) {
        c.0 = t;
    }
}
```

我们现在可以为这些函数创建 `use fun` 别名：

```move
module b::example {
    use fun a::cup::cup_borrow as Cup.borrow;
    use fun a::cup::cup_value as Cup.value;
    use fun a::cup::cup_swap as Cup.set;

    fun example(c: &mut Cup<u64>) {
        let _ = c.borrow(); // 解析为 a::cup::cup_borrow
        let v = c.value(); // 解析为 a::cup::cup_value
        c.set(v * 2); // 解析为 a::cup::cup_swap
    }
}
```

注意，`use fun` 中的 `<function>` 不必是一个完全解析的路径，可以使用别名，因此上述示例中的声明也可以等效地写成：

```move
    use a::cup::{Self, cup_swap};

    use fun cup::cup_borrow as Cup.borrow;
    use fun cup::cup_value as Cup.value;
    use fun cup_swap as Cup.set;
```

虽然这些示例在当前模块中重命名函数很有趣，但该功能在声明其他模块中的类型的方法时可能更有用。例如，如果我们想向 `Cup` 添加一个新实用程序，我们可以使用 `use fun` 别名并仍然使用方法语法：

```move
module b::example {

    fun double(c: &Cup<u64>): Cup<u64> {
        let v = c.value();
        Cup::new(v * 2)
    }

}
```

通常，我们不得不调用 `double(&c)`，因为 `b::example` 并未定义 `Cup`，但我们可以使用 `use fun` 别名：

```move
    fun double_double(c: Cup<u64>): (Cup<u64>, Cup<u64>) {
        use fun b::example::double as Cup.dub;
        (c.dub(), c.dub()) // 在两个调用中都解析为 b::example::double
    }
```

虽然 `use fun` 可以在任何范围内创建，但 `use fun` 的目标 `<function>` 必须具有与 `<type>` 相同的第一个参数。

```move
public struct X() has copy, drop, store;

fun new(): X { X() }
fun flag(flag: bool): u8 { if (flag) 1 else 0 }

use fun new as X.new; // 错误!
use fun flag as X.flag; // 错误!
// `new` 和 `flag` 都没有第一个参数为类型 `X`
```

但 `<type>` 的任何第一个参数都可以使用，包括引用和可变引用：

```move
public struct X() has copy, drop, store;

public fun by_val(_: X) {}
public fun by_ref(_: &X) {}
public fun by_mut(_: &mut X) {}

// 全部3个有效，在任何范围内
use fun by_val as X.v;
use fun by_ref as X.r;
use fun by_mut as X.m;
```

注意对于泛型，方法是关联到该泛型类型的所有实例的。不能根据实例化情况重载方法以解析为不同的函数。

```move
public struct Cup<T>(T) has copy, drop, store;

public fun value<T: copy>(c: &Cup<T>): T {
    c.0
}

use fun value as Cup<bool>.flag; // 错误!
use fun value as Cup<u64>.num; // 错误!
// 在这两种情况下，`use fun` 别名不能是泛型的，它们必须对该类型的所有实例有效
```

### `public use fun` 别名

与传统的 [`use`](uses.md) 不同，`use fun` 语句可以被声明为 `public`，允许在其声明的作用域之外使用。如果在定义接收者类型的模块中声明，`use fun` 可以是 `public` 的，这类似于在定义模块中为函数[自动创建](#functions-in-the-defining-module)的方法别名。或者可以认为，对于每个在定义模块中定义的第一个参数为接收者类型的函数，自动创建一个隐式的 `public use fun`。这两种观点是等价的。

```move
module a::cup {
    public struct Cup<T>(T) has copy, drop, store;

    public use fun cup_borrow as Cup.borrow;
    public fun cup_borrow<T>(c: &Cup<T>): &T {
        &c.0
    }
}
```

在此示例中，为 `a::cup::Cup.borrow` 和 `a::cup::Cup.cup_borrow` 创建了一个公共方法别名。两者都解析为 `a::cup::cup_borrow`，并且两者都是“公共的”，这意味着它们可以在 `a::cup` 之外使用，而无需额外的 `use` 或 `use fun`。

```move
module b::example {

    fun example<T: drop>(c: a::cup::Cup<u64>) {
        c.borrow(); // 解析为 a::cup::cup_borrow
        c.cup_borrow(); // 解析为 a::cup::cup_borrow
    }
}
```

`public use fun` 声明因此作为一种重命名函数的方法，如果你想为方法语法提供一个更简洁的名称。这在具有多种类型且每种类型具有类似名称的函数的模块中特别有用。

```move
module a::shapes {

    public struct Rectangle { base: u64, height: u64 }
    public struct Box { base: u64, height: u64, depth: u64 }

    // Rectangle 和 Box 可以具有相同名称的方法

    public use fun rectangle_base as Rectangle.base;
    public fun rectangle_base(rectangle: &Rectangle): u64 {
        rectangle.base
    }

    public use fun box_base as Box.base;
    public fun box_base(box: &Box): u64 {
        box.base
    }

}
```

`public use fun` 的另一个用途是为其他模块中的类型添加方法。这在单个包中跨多个模块分散的函数中很有用。

```move
module a::cup {
    public struct Cup<T>(T) has copy, drop, store;

    public fun new<T>(t: T): Cup<T> { Cup(t) }
    public fun borrow<T>(c: &Cup<T>): &T {
        &c.0
    }
    // `public use fun` 引用在另一个模块中定义的函数
    public use fun a::utils::split as Cup.split;
}

module a::utils {
    use a::m::{Self, Cup};

    public fun split<u64>(c: Cup<u64>): (Cup<u64>, Cup<u64>) {
        let Cup(t) = c;
        let half = t / 2;
        let rem = if (t > 0) t - half else 0;
        (cup::new(half), cup::new(rem))
    }

}
```

需要注意的是，这个 `public use fun` 不会创建循环依赖，因为 `use fun` 在模块编译后不存在--所有方法都是静态解析的。

### 与 `use` 别名的交互

需要注意的一点是，方法别名遵循正常的 `use` 别名规则。

```move
module a::cup {
    public struct Cup<T>(T) has copy, drop, store;

    public fun cup_borrow<T>(c: &Cup<T>): &T {
        &c.0
    }
}

module b::other {
    use a::cup::{Cup, cup_borrow as borrow};

    fun example(c: &Cup<u64>) {
        c.borrow(); // 解析为 a::cup::cup_borrow
    }
}
```

一个有用的思路是，`use` 会在可能的情况下为函数创建一个隐式的 `use fun` 别名。在这种情况下，`use a::cup::cup_borrow as borrow` 创建了一个隐式的 `use fun a::cup::cup_borrow as Cup.borrow`，因为它是一个有效的 `use fun` 别名。这两种观点是等价的。这种推理可以帮助理解特定方法在遮蔽情况下如何解析。有关更多详细信息，请参见[作用域](#scoping)中的案例。

### 作用域

如果不是 `public` 的，`use fun` 别名是其作用域的局部变量，就像普通的[`use`](uses.md)一样。例如：

```move
module a::m {
    public struct X() has copy, drop, store;
    public fun foo(_: &X) {}
    public fun bar(_: &X) {}
}

module b::other {
    use a::m::X;

    use fun a::m::foo as X.f;

    fun example(x: &X) {
        x.f(); // 解析为 a::m::foo
        {
            use a::m::bar as f;
            x.f(); // 解析为 a::m::bar
        };
        x.f(); // 仍然解析为 a::m::foo
        {
            use fun a::m::bar as X.f;
            x.f(); // 解析为 a::m::bar
        }
    }
```

## 自动借用

在解析方法时，编译器会在函数需要引用时自动借用接收者。例如：

```move
module a::m {
    public struct X() has copy, drop;
    public fun by_val(_: X) {}
    public fun by_ref(_: &X) {}
    public fun by_mut(_: &mut X) {}

    fun example(mut x: X) {
        x.by_ref(); // 解析为 a::m::by_ref(&x)
        x.by_mut(); // 解析为 a::m::by_mut(&mut x)
    }
}
```

在这些示例中，`x` 被自动借用为 `&x` 和 `&mut x`。这也适用于字段访问：

```move
module a::m {
    public struct X() has copy, drop;
    public fun by_val(_: X) {}
    public fun by_ref(_: &X) {}
    public fun by_mut(_: &mut X) {}

    public struct Y has drop { x: X }

    fun example(mut y: Y) {
        y.x.by_ref(); // 解析为 a::m::by_ref(&y.x)
        y.x.by_mut(); // 解析为 a::m::by_mut(&mut y.x)
    }
}
```

请注意，在这两个示例中，本地变量必须标记为 [`mut`](./variables.md) 以允许 `&mut` 借用。否则，会出现错误，提示 `x`（或第二个示例中的 `y`）不可变。

需要记住的是，如果没有引用，正常的变量和字段访问规则将生效。这意味着如果没有借用，值可能会被移动或复制。

```move
module a::m {
    public struct X() has copy, drop;
    public fun by_val(_: X) {}
    public fun by_ref(_: &X) {}
    public fun by_mut(_: &mut X) {}

    public struct Y has drop { x: X }
    public fun drop_y(y: Y) { y }

    fun example(y: Y) {
        y.x.by_val(); // 复制 `y.x` 因为 `by_val` 是按值传递的，并且 `X` 具有 `copy`
        y.drop_y(); // 移动 `y` 因为 `drop_y` 是按值传递的，并且 `Y` 没有 `copy`
    }
}
```

## 链式调用

方法调用可以是链式调用，因为任何表达式都可以是方法的接收者。

```move
module a::shapes {
    public struct Point has copy, drop, store { x: u64, y: u64 }
    public struct Line has copy, drop, store { start: Point, end: Point }

    public fun x(p: &Point): u64 { p.x }
    public fun y(p: &Point): u64 { p.y }

    public fun start(l: &Line): &Point { &l.start }
    public fun end(l: &Line): &Point { &l.end }

}

module b::example {
    use a::shapes::Line;

    public fun x_values(l: Line): (u64, u64) {
        (l.start().x(), l.end().x())
    }

}
```

在这个例子中，对于 `l.start().x()`，编译器首先将 `l.start()` 解析为 `a::shapes::start(&l)`。然后将 `.x()` 解析为 `a::shapes::x(a::shapes::start(&l))`。同样适用于 `l.end().x()`。请记住，这个功能并不是"特别的"--`.` 左边的表达式可以是任何表达式，编译器将正常解析方法调用。我们特别提到这种"链式调用"，因为它是增加可读性的常见做法。
