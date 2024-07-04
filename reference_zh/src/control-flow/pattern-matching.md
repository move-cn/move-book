# 模式匹配

`match` 表达式是一种强大的控制结构，允许你将一个值与一系列模式进行比较，然后根据首先匹配的模式执行代码。模式可以是简单的字面量，也可以是复杂的、嵌套的结构体和枚举定义。与基于 `bool` 类型测试表达式的 `if` 表达式不同，`match` 表达式操作任何类型的值并选择多个分支之一。

`match` 表达式可以匹配 Move 值以及可变或不可变引用，并相应地绑定子模式。

例如：

```move
fun run(x: u64): u64 {
    match (x) {
        1 => 2,
        2 => 3,
        x => x,
    }
}

run(1); // 返回 2
run(2); // 返回 3
run(3); // 返回 3
run(0); // 返回 0
```

## `match` 语法

`match` 包含一个表达式和一个非空的 _match 分支_ 列表，用逗号分隔。

每个 match 分支由一个模式 (`p`)、一个可选的守卫 (`if (g)`，其中 `g` 是一个 `bool` 类型的表达式)、一个箭头 (`=>`) 和一个分支表达式 (`e`) 组成，当模式匹配时执行。例如，

```move
match (expression) {
    pattern1 if (guard_expression) => expression1,
    pattern2 => expression2,
    pattern3 => { expression3, expression4, ... },
}
```

match 分支按顺序从上到下检查，第一个匹配的模式（如果存在，则匹配的守卫表达式为 `true`）将被执行。

请注意，`match` 中的分支必须是穷尽的，意味着匹配的类型的每一个可能的值都必须由 `match` 中的一个模式覆盖。如果分支不穷尽，编译器将抛出错误。

## 模式语法

一个模式与一个值匹配，如果该值等于模式，其中变量和通配符（例如 `x`、`y`、`_` 或 `..`）与任何值“相等”。

模式用于匹配值。模式可以是：

| 模式              | 描述                                                                |
| ----------------- | ------------------------------------------------------------------- |
| 字面量            | 字面量值，例如 `1`、`true`、`@0x1`                                  |
| 常量              | 常量值，例如 `MyConstant`                                           |
| 变量              | 变量，例如 `x`、`y`、`z`                                            |
| 通配符            | 通配符，例如 `_`                                                    |
| 构造器            | 构造器模式，例如 `MyStruct { x, y }`、`MyEnum::Variant(x)`           |
| @ 模式            | at 模式，例如 `x @ MyEnum::Variant(..)`                             |
| 或模式            | 或模式，例如 `MyEnum::Variant(..) \| MyEnum::OtherVariant(..)`       |
| 多元通配符        | 多元通配符，例如 `MyEnum::Variant(..)`                              |
| 可变绑定          | 可变绑定模式，例如 `mut x`                                          |

Move 中的模式具有以下语法：

```bnf
pattern = <literal>
        | <constant>
        | <variable>
        | _
        | C { <variable> : inner-pattern ["," <variable> : inner-pattern]* } // C 是结构体或枚举变体
        | C ( inner-pattern ["," inner-pattern]* ... )                       // C 是结构体或枚举变体
        | C                                                                  // C 是枚举变体
        | <variable> @ top-level-pattern
        | pattern | pattern
        | mut <variable>
inner-pattern = pattern
              | ..     // 多元通配符
```

一些模式示例：

```move
// 字面量模式
1

// 常量模式
MyConstant

// 变量模式
x

// 通配符模式
_

// 构造器模式，匹配 `MyEnum::Variant`，字段为 `1` 和 `true`
MyEnum::Variant(1, true)

// 构造器模式，匹配 `MyEnum::Variant`，字段为 `1` 并将第二个字段的值绑定到 `x`
MyEnum::Variant(1, x)

// 多元通配符模式，匹配 `MyEnum::Variant` 变体中的多个字段
MyEnum::Variant(..)

// 构造器模式，匹配 `MyStruct` 的 `x` 字段并将 `y` 字段绑定到 `other_variable`
MyStruct { x, y: other_variable }

// at 模式，匹配 `MyEnum::Variant` 并将整个值绑定到 `x`
x @ MyEnum::Variant(..)

// 或模式，匹配 `MyEnum::Variant` 或 `MyEnum::OtherVariant`
MyEnum::Variant(..) | MyEnum::OtherVariant(..)

// 与上述或模式相同，但使用显式通配符
MyEnum::Variant(_, _) | MyEnum::OtherVariant(_, _)

// 或模式，匹配 `MyEnum::Variant` 或 `MyEnum::OtherVariant`，并将 u64 字段绑定到 `x`
MyEnum::Variant(x, _) | MyEnum::OtherVariant(_, x)

// 构造器模式，匹配 `OtherEnum::V` 并且内部 `MyEnum` 是 `MyEnum::Variant`
OtherEnum::V(MyEnum::Variant(..))
```

### 模式和变量

包含变量的模式将变量绑定到匹配的值或值的子组件。这些变量可以在任何匹配守卫表达式中使用，也可以在匹配分支的右侧使用。例如：

```move
public struct Wrapper(u64)

fun add_under_wrapper_unless_equal(wrapper: Wrapper, x: u64): u64 {
    match (wrapper) {
        Wrapper(y) if (y == x) => Wrapper(y),
        Wrapper(y) => y + x,
    }
}
add_under_wrapper_unless_equal(Wrapper(1), 2); // 返回 Wrapper(3)
add_under_wrapper_unless_equal(Wrapper(2), 3); // 返回 Wrapper(5)
add_under_wrapper_unless_equal(Wrapper(3), 3); // 返回 Wrapper(3)
```

### 组合模式

模式可以嵌套，但也可以使用或运算符 (`|`) 组合模式。例如，`p1 | p2` 成功匹配如果模式 `p1` 或 `p2` 中的任何一个匹配。这种模式可以出现在任何地方——作为顶层模式或另一个模式中的子模式。

```move
public enum MyEnum has drop {
    Variant(u64, bool),
    OtherVariant(bool, u64),
}

fun test_or_pattern(x: u64): u64 {
    match (x) {
        MyEnum::Variant(1 | 2 | 3, true) | MyEnum::OtherVariant(true, 1 | 2 | 3) => 1,
        MyEnum::Variant(8, true) | MyEnum::OtherVariant(_, 6 | 7) => 2,
        _ => 3,
    }
}

test_or_pattern(MyEnum::Variant(3, true)); // 返回 1
test_or_pattern(MyEnum::OtherVariant(true, 2)); // 返回 1
test_or_pattern(MyEnum::Variant(8, true)); // 返回 2
test_or_pattern(MyEnum::OtherVariant(false, 7)); // 返回 2
test_or_pattern(MyEnum::OtherVariant(false, 80)); // 返回 3
```

### 某些模式的限制

`mut` 和 `..` 模式在使用时有特定的条件，如 [特定模式的限制](#limitations-on-specific-patterns) 中所述。大体上，`mut` 修饰符只能用于变量模式，`..` 模式只能在构造器模式中使用一次——不能作为顶层模式使用。

以下是 `..` 模式的 _无效_ 用法，因为它用作顶层模式：

```move
match (x) {
    .. => 1,
    // 错误: `..` 模式只能在构造器模式中使用
}

match (x) {
    MyStruct(.., ..) => 1,
    // 错误:    ^^  `..` 模式只能在构造器模式中使用一次
}
```

### 模式类型

模式不是表达式，但它们仍然是类型化的。这意味着模式的类型必须与匹配值的类型匹配。例如，模式 `1` 具有整数类型，模式 `MyEnum::Variant(1, true)` 具有 `MyEnum` 类型，模式 `MyStruct { x, y }` 具有 `MyStruct` 类型，而 `OtherStruct<bool> { x: true, y: 1}` 具有 `OtherStruct<bool>` 类型。如果尝试匹配类型与模式类型不同的表达式，将导致类型错误。例如：

```move
match (1) {
    // `true` 字面量模式是 `bool` 类型，因此这是一个类型错误。
    true => 1,
    // 类型错误: 预期类型 u64，找到 bool
    _ => 2,
}
```

同样，以下也会导致类型错误，因为 `MyEnum` 和 `MyStruct` 是不同的类型：

```
match (MyStruct { x: 0, y: 0 }) {
    MyEnum::Variant(..) => 1,
    // TYPE ERROR: expected type MyEnum, found MyStruct
}
```

## 匹配

在深入探讨模式匹配的具体细节以及一个值与模式“匹配”意味着什么之前，让我们通过几个示例来提供一个直观的理解。

```move
fun test_lit(x: u64): u8 {
    match (x) {
        1 => 2,
        2 => 3,
        _ => 4,
    }
}
test_lit(1); // 返回 2
test_lit(2); // 返回 3
test_lit(3); // 返回 4
test_lit(10); // 返回 4

fun test_var(x: u64): u64 {
    match (x) {
        y => y,
    }
}
test_var(1); // 返回 1
test_var(2); // 返回 2
test_var(3); // 返回 3
...

const MyConstant: u64 = 10;
fun test_constant(x: u64): u64 {
    match (x) {
        MyConstant => 1,
        _ => 2,
    }
}
test_constant(MyConstant); // 返回 1
test_constant(10); // 返回 1
test_constant(20); // 返回 2

fun test_or_pattern(x: u64): u64 {
    match (x) {
        1 | 2 | 3 => 1,
        4 | 5 | 6 => 2,
        _ => 3,
    }
}
test_or_pattern(3); // 返回 1
test_or_pattern(5); // 返回 2
test_or_pattern(70); // 返回 3

fun test_or_at_pattern(x: u64): u64 {
    match (x) {
        x @ (1 | 2 | 3) => x + 1,
        y @ (4 | 5 | 6) => y + 2,
        z => z + 3,
    }
}
test_or_pattern(2); // 返回 3
test_or_pattern(5); // 返回 7
test_or_pattern(70); // 返回 73
```

从这些示例中最重要的一点是，一个模式匹配一个值如果该值等于该模式，并且通配符/变量模式匹配任何值。这对文字、变量和常量都是如此。例如，在 `test_lit` 函数中，值 `1` 匹配模式 `1`，值 `2` 匹配模式 `2`，而值 `3` 匹配通配符 `_`。类似地，在 `test_var` 函数中，值 `1` 和 `2` 都匹配模式 `y`。

变量 `x` 匹配（或“等于”）任何值，而通配符 `_` 匹配任何值（但只匹配一个值）。或者模式就像一个逻辑 OR，值匹配模式如果它匹配或模式中的任何一个模式，所以 `p1 | p2 | p3` 应该被解读为“匹配 p1，或 p2，或 p3”。

### 匹配构造函数

模式匹配包括构造函数模式的概念。这些模式允许你检查并访问结构体和枚举中的深层次内容，是模式匹配最强大的部分之一。构造函数模式与变量绑定结合，允许你通过结构匹配值，并提取你关心的部分以在匹配分支的右侧使用。

看看下面的例子：

```move
public enum MyEnum has drop {
    Variant(u64, bool),
    OtherVariant(bool, u64),
}

fun f(x: MyEnum): u64 {
    match (x) {
        MyEnum::Variant(1, true) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        MyEnum::Variant(..) => 3,
        MyEnum::OtherVariant(..) => 4,
    }
}
f(MyEnum::Variant(1, true)); // 返回 1
f(MyEnum::Variant(2, true)); // 返回 3
f(MyEnum::OtherVariant(false, 3)); // 返回 2
f(MyEnum::OtherVariant(true, 3)); // 返回 2
f(MyEnum::OtherVariant(true, 2)); // 返回 4
```

这段代码的意思是“如果 `x` 是 `MyEnum::Variant` 并且字段是 `1` 和 `true`，则返回 `1`。如果它是 `MyEnum::OtherVariant` 并且第一个字段是任何值，第二个字段是 `3`，则返回 `2`。如果它是 `MyEnum::Variant` 并且字段是任何值，则返回 `3`。最后，如果它是 `MyEnum::OtherVariant` 并且字段是任何值，则返回 `4`”。

你还可以嵌套模式。因此，如果你想匹配 1、2 或 10，而不仅仅是匹配前面的 `MyEnum::Variant` 中的 1，你可以使用 or 模式来实现：

```move
fun f(x: MyEnum): u64 {
    match (x) {
        MyEnum::Variant(1 | 2 | 10, true) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        MyEnum::Variant(..) => 3,
        MyEnum::OtherVariant(..) => 4,
    }
}
f(MyEnum::Variant(1, true)); // 返回 1
f(MyEnum::Variant(2, true)); // 返回 1
f(MyEnum::Variant(10, true)); // 返回 1
f(MyEnum::Variant(10, false)); // 返回 3
```

### 能力约束

此外，匹配绑定受到与Move其他方面相同的能力限制。特别是，如果你试图使用通配符匹配一个没有`drop`能力的值（非引用），编译器将发出错误信号，因为通配符期望丢弃该值。同样，如果你使用绑定器绑定一个非`drop`值，它必须在匹配分支的右侧使用。此外，如果你完全解构该值，你就拆解了它，这与[非`drop`结构体拆解](../structs.md#destroying-structs-via-pattern-matching)的语义相匹配。有关`drop`能力的更多详细信息，请参阅[能力部分的`drop`](../abilities.md#drop)。

```move
public struct NonDrop(u64)

fun drop_nondrop(x: NonDrop) {
    match (x) {
        NonDrop(1) => 1,
        _ => 2
        // 错误：不能在不可丢弃的值上使用通配符匹配
    }
}

fun destructure_nondrop(x: NonDrop) {
    match (x) {
        NonDrop(1) => 1,
        NonDrop(_) => 2
        // 正确！
    }
}

fun use_nondrop(x: NonDrop): NonDrop {
    match (x) {
        NonDrop(1) => NonDrop(8),
        x => x
    }
}
```

## 穷尽性

Move中的`match`表达式必须是_穷尽的_：被匹配类型的每个可能值都必须被匹配分支中的一个模式所覆盖。如果匹配分支序列不是穷尽的，编译器将引发错误。注意，任何带有守卫表达式的分支都不会贡献于匹配穷尽性，因为它可能在运行时匹配失败。

例如，对`u8`的匹配只有在匹配0到255（包括255）的_每个_数字时才是穷尽的，除非存在通配符或变量模式。同样，对`bool`的匹配需要匹配`true`和`false`两者，除非存在通配符或变量模式。

对于结构体，因为类型只有一种构造函数，所以只需要匹配一个构造函数，但结构体内的字段也需要被穷尽地匹配。相反，枚举可能定义多个变体，每个变体都必须被匹配（包括任何子字段）才能被视为穷尽匹配。

因为下划线和变量是匹配任何内容的通配符，它们被视为匹配该位置上它们所匹配类型的所有值。此外，多元通配符模式`..`可用于匹配结构体或枚举变体内的多个值。

看一些_非穷尽_匹配的例子，考虑以下内容：

```move
public enum MyEnum {
    Variant(u64, bool),
    OtherVariant(bool, u64),
}

public struct Pair<T>(T, T)

fun f(x: MyEnum): u8 {
    match (x) {
        MyEnum::Variant(1, true) => 1,
        MyEnum::Variant(_, _) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        // 错误：不穷尽，因为值`MyEnum::OtherVariant(_, 4)`没有被匹配。
    }
}

fun match_pair_bool(x: Pair<bool>): u8 {
    match (x) {
        Pair(true, true) => 1,
        Pair(true, false) => 1,
        Pair(false, false) => 1,
        // 错误：不穷尽，因为值`Pair(false, true)`没有被匹配。
    }
}
```

可以通过在匹配分支末尾添加通配符模式，或完全匹配剩余值来使这些例子变得穷尽：

```move
fun f(x: MyEnum): u8 {
    match (x) {
        MyEnum::Variant(1, true) => 1,
        MyEnum::Variant(_, _) => 1,
        MyEnum::OtherVariant(_, 3) => 2,
        // 现在穷尽了，因为这将匹配MyEnum::OtherVariant的所有值
        MyEnum::OtherVariant(..) => 2,
    }
}

fun match_pair_bool(x: Pair<bool>): u8 {
    match (x) {
        Pair(true, true) => 1,
        Pair(true, false) => 1,
        Pair(false, false) => 1,
        // 现在穷尽了，因为这将匹配Pair<bool>的所有值
        Pair(false, true) => 1,
    }
}
```

## 守卫

如前所述，你可以通过在模式后添加一个 `if` 子句来向匹配分支添加守卫。这个守卫将在模式匹配之后但在箭头右侧的表达式求值之前运行。如果守卫表达式求值为 `true`，则箭头右侧的表达式将被求值；如果求值为 `false`，则将被视为匹配失败，并检查 `match` 表达式中的下一个匹配分支。

```move
fun match_with_guard(x: u64): u64 {
    match (x) {
        1 if (false) => 1,
        1 => 2,
        _ => 3,
    }
}

match_with_guard(1); // 返回 2
match_with_guard(0); // 返回 3
```

守卫表达式可以引用在模式求值期间绑定的变量。然而，请注意，无论模式是如何匹配的，变量在守卫中始终仅作为不可变引用可用——即使变量上有可变性说明符或者模式是按值匹配的。

```move
fun incr(x: &mut u64) {
    *x = *x + 1;
}

fun match_with_guard_incr(x: u64): u64 {
    match (x) {
        x if ({ incr(&mut x); x == 1 }) => 1,
        // 错误:    ^^^ 对不可变值的无效借用
        _ => 2,
    }
}

fun match_with_guard_incr2(x: &mut u64): u64 {
    match (x) {
        x if ({ incr(&mut x); x == 1 }) => 1,
        // 错误:    ^^^ 对不可变值的无效借用
        _ => 2,
    }
}
```

此外，需要注意的是，任何具有守卫表达式的匹配分支都不会被视为出于穷尽性检查的目的，因为编译器无法静态地评估守卫表达式。

## 特定模式的限制

在模式中使用 `..` 和 `mut` 模式修饰符时存在一些限制。

### 可变性使用

`mut` 修饰符可以放在变量模式上，以指定该变量在匹配分支的右侧表达式中是可变的。请注意，由于 `mut` 修饰符仅表示变量是可变的，而不是底层数据，因此可以在所有类型的匹配中使用（按值、不可变引用和可变引用）。

请注意，`mut` 修饰符只能应用于变量，而不能应用于其他类型的模式。

```move
public struct MyStruct(u64);

fun top_level_mut(x: MyStruct) {
    match (x) {
        mut MyStruct(y) => 1,
        // 错误: 不能在非变量模式上使用 mut
    }
}

fun mut_on_immut(x: &MyStruct): u64 {
    match (x) {
        MyStruct(mut y) => {
            y = &(*y + 1);
            *y
        }
    }
}

fun mut_on_value(x: MyStruct): u64 {
    match (x) {
        MyStruct(mut y) =>  {
            *y = *y + 1;
            *y
        },
    }
}

fun mut_on_mut(x: &mut MyStruct): u64 {
    match (x) {
        MyStruct(mut y) =>  {
            *y = *y + 1;
            *y
        },
    }
}

let mut x = MyStruct(1);

mut_on_mut(&mut x); // 返回 2
x.0; // 返回 2

mut_on_immut(&x); // 返回 3
x.0; // 返回 2

mut_on_value(x); // 返回 3
```

### `..` 的使用

`..` 模式只能在构造函数模式中用作通配符，匹配任意数量的字段——编译器将 `..` 展开为在构造函数模式中插入任何缺失字段中的 `_`。所以 `MyStruct(_, _, _)` 等同于 `MyStruct(..)`，`MyStruct(1, _, _)` 等同于 `MyStruct(1, ..)`。因此，对于 `..` 模式的使用有一些限制：

- 它只能在构造函数模式中使用**一次**；
- 在位置参数中，它可以在构造函数模式中的开始、中间或结尾使用；
- 在命名参数中，它只能在构造函数模式的结尾使用；

```move
public struct MyStruct(u64, u64, u64, u64) has drop;

public struct MyStruct2 {
    x: u64,
    y: u64,
    z: u64,
    w: u64,
}

fun wild_match(x: MyStruct) {
    match (x) {
        MyStruct(.., 1) => 1,
        // OK! `..` 模式可以在构造函数模式的开头使用
        MyStruct(1, ..) => 2,
        // OK! `..` 模式可以在构造函数模式的结尾使用
        MyStruct(1, .., 1) => 3,
        // OK! `..` 模式可以在构造函数模式的中间使用
        MyStruct(1, .., 1, 1) => 4,
        MyStruct(..) => 5,
    }
}

fun wild_match2(x: MyStruct2) {
    match (x) {
        MyStruct2 { x: 1, .. } => 1,
        MyStruct2 { x: 1, w: 2 .. } => 2,
        MyStruct2 { .. } => 3,
    }
}
```