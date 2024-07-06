# Functions（函数）

函数被声明在模块内部，并定义模块的逻辑和行为。函数可以被重用，可以作为其他函数的调用点或作为执行的入口点。

## 声明

函数使用 `fun` 关键字声明，后面跟着函数名、类型参数、参数列表、返回类型和函数体。

```text
<visibility>? <entry>? fun <identifier><[type_parameters: constraint],*>([identifier: type],*): <return_type> <function_body>
```

例如

```move
fun foo<T1, T2>(x: u64, y: T1, z: T2): (T2, T1, u64) { (z, y, x) }
```

### 可见性

模块内的函数默认只能在同一模块内调用。这些内部（有时称为私有）函数不能从其他模块调用或作为入口点。

```move
module a::m {
    fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 合法
}

module b::other {
    fun calls_m_foo(): u64 {
        a::m::foo() // 错误！
//      ^^^^^^^^^^^ 'foo' 是 'a::m' 内部的
    }
}
```

要允许从其他模块访问该函数，函数必须声明为 `public` 或 `public(package)`。与可见性相对应，一个 [`entry`](#entry-modifier) 函数可以作为执行的入口点。

#### `public` 可见性

`public` 函数可以被**任何**模块中的**任何**函数调用。如下示例所示，`public` 函数可以被：

- 同一模块中定义的其他函数调用，
- 另一个模块中定义的函数调用，或
- 作为执行的入口点。

```move
module a::m {
    public fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 合法
}

module b::other {
    fun calls_m_foo(): u64 {
        a::m::foo() // 合法
    }
}
```

有关入口点执行的更多详细信息，请参阅[下面的章节](#entry-modifier)。

#### `public(package)` 可见性

`public(package)` 可见性修饰符是 `public` 修饰符的一种更受限制的形式，用于更精确地控制函数的使用范围。`public(package)` 函数可以被：

- 同一模块中定义的其他函数调用，
- 同一包（同一地址）中定义的其他函数调用。

```move
module a::m {
    public(package) fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 合法
}

module a::n {
    fun calls_m_foo(): u64 {
        a::m::foo() // 合法，在 `a` 中也可以调用
    }
}

module b::other {
    fun calls_m_foo(): u64 {
        b::m::foo() // 错误！
//      ^^^^^^^^^^^ 'foo' 只能从 `a` 包中的模块调用
    }
}
```

#### 已废弃的 `public(friend)` 可见性

在引入 `public(package)` 之前，`public(friend)` 用于允许有限的公共访问权限，但必须由被调用模块显式列出允许的模块列表。详细信息请参阅 [Friends](./friends.md)。

### `entry` 修饰符

除了 `public` 函数外，可能还有一些函数希望用作执行的入口点。`entry` 修饰符设计用于允许模块函数发起执行，而无需将功能公开给其他模块。

虽然 `entry` 函数可以作为 Move 程序的起始点，但它们并不限制于此用例。

例如：

```move
module a::m {
    entry fun foo(): u64 { 0 }
    fun calls_foo(): u64 { foo() } // 合法！
}

module a::n {
    fun calls_m_foo(): u64 {
        a::m::foo() // 错误！
//      ^^^^^^^^^^^ 'foo' 是 'a::m' 内部的
    }
}
```

`entry` 函数可能对其参数和返回类型有限制。但这些限制是针对每个 Move 部署的特定情况的。

关于 Sui 中 `entry` 函数的文档可以在 [这里找到](https://docs.sui.io/concepts/sui-move-concepts/entry-functions)。

为了更轻松地进行测试，`entry` 函数可以从 [`#[test]` 和 `#[test_only]`](./unit-testing.md) 上下文中调用。

```move
module a::m {
    entry fun foo(): u64 { 0 }
}
module a::m_test {
    #[test]
    fun my_test(): u64 { a::m::foo() } // 合法！
    #[test_only]
    fun my_test_helper(): u64 { a::m::foo() } // 合法！
}
```

### 名称

函数名称可以以字母 `a` 到 `z` 开头。在第一个字符之后，函数名称可以包含下划线 `_`，字母 `a` 到 `z`，字母 `A` 到 `Z`，或数字 `0` 到 `9`。

```move
fun fOO() {}
fun bar_42() {}
fun bAZ_19() {}
```

### 类型参数

在名称之后，函数可以有类型参数

```move
fun id<T>(x: T): T { x }
fun example<T1: copy, T2>(x: T1, y: T2): (T1, T1, T2) { (copy x, x, y) }
```

更多详情请参阅 [Move generics](./generics.md)。

### 参数

函数参数通过本地变量名后跟类型注解声明

```move
fun add(x: u64, y: u64): u64 { x + y }
```

我们将其读作 `x` 具有类型 `u64`

一个函数也可以完全没有任何参数。

```move
fun useless() { }
```

这对于创建新的或空的数据结构非常常见。

```move
module a::example {
  public struct Counter { count: u64 }

  fun new_counter(): Counter {
      Counter { count: 0 }
  }
}
```

### 返回类型

在参数之后，函数指定其返回类型。

```move
fun zero(): u64 { 0 }
```

这里的 `: u64` 表示函数的返回类型是 `u64`。

使用 [元组](./primitive-types/tuples.md)，函数可以返回多个值：

```move
fun one_two_three(): (u64, u64, u64) { (0, 1, 2) }
```

如果没有指定返回类型，函数具有隐式的返回类型单元 `()`。以下函数是等效的：

```move
fun just_unit(): () { () }
fun just_unit() { () }
fun just_unit() { }
```

正如在 [元组部分](./primitive-types/tuples.md) 提到的，这些元组 "值" 在运行时并不存在。这意味着返回单元 `()` 的函数在执行期间不返回任何值。

### 函数体

函数体是一个表达式块。函数的返回值是序列中的最后一个值

```move
fun example(): u64 {
    let x = 0;
    x = x + 1;
    x // 返回 'x'
}
```

有关返回值的更多信息，请参阅 [下面的章节](#returning-values)。

有关表达式块的更多信息，请参阅 [Move variables](./variables.md)。

### 原生函数

某些函数没有指定函数体,而是由虚拟机(VM)提供函数体。这些函数被标记为`native`。

如果不修改VM源代码,程序员无法添加新的原生函数。此外,`native`函数的目的是用于标准库代码或特定Move环境所需的功能。

你可能会看到的大多数`native`函数都在标准库代码中,例如`vector`:

```move
module std::vector {
    native public fun length<Element>(v: &vector<Element>): u64;
    ...
}
```

## 函数调用

调用函数时,可以通过别名或完全限定名来指定函数名:

```move
module a::example {
    public fun zero(): u64 { 0 }
}

module b::other {
    use a::example::{Self, zero};
    fun call_zero() {
        // 有了上面的`use`语句,以下所有调用都是等价的
        a::example::zero();
        example::zero();
        zero();
    }
}
```

调用函数时,必须为每个参数提供一个实参。

```move
module a::example {
    public fun takes_none(): u64 { 0 }
    public fun takes_one(x: u64): u64 { x }
    public fun takes_two(x: u64, y: u64): u64 { x + y }
    public fun takes_three(x: u64, y: u64, z: u64): u64 { x + y + z }
}

module b::other {
    fun call_all() {
        a::example::takes_none();
        a::example::takes_one(0);
        a::example::takes_two(0, 1);
        a::example::takes_three(0, 1, 2);
    }
}
```

类型参数可以被显式指定或推断。以下两个调用是等价的:

```move
module aexample {
    public fun id<T>(x: T): T { x }
}

module b::other {
    fun call_all() {
        a::example::id(0);
        a::example::id<u64>(0);
    }
}
```

更多详情,请参见[Move泛型](./generics.md)。

## 返回值

函数的结果,即"返回值",是其函数体的最终值。例如:

```move
fun add(x: u64, y: u64): u64 {
    x + y
}
```

这里的返回值是`x + y`的结果。

[如上所述](#function-body),函数的主体是一个[表达式块](./variables.md)。表达式块可以按顺序执行各种语句,块中的最后一个表达式将成为该块的值:

```move
fun double_and_add(x: u64, y: u64): u64 {
    let double_x = x * 2;
    let double_y = y * 2;
    double_x + double_y
}
```

这里的返回值是`double_x + double_y`的结果。

### `return`表达式

函数隐式地返回其主体计算的值。但是,函数也可以使用显式的`return`表达式:

```move
fun f1(): u64 { return 0 }
fun f2(): u64 { 0 }
```

这两个函数是等价的。在这个稍微复杂一点的例子中,函数将两个`u64`值相减,但如果第二个值太大,则提前返回`0`:

```move
fun safe_sub(x: u64, y: u64): u64 {
    if (y > x) return 0;
    x - y
}
```

注意,这个函数的主体也可以写成`if (y > x) 0 else x - y`。

然而,`return`的真正优势在于可以在其他控制流结构的深处退出。在这个例子中,函数遍历一个向量以查找给定值的索引:

```move
use std::vector;
use std::option::{Self, Option};
fun index_of<T>(v: &vector<T>, target: &T): Option<u64> {
    let i = 0;
    let n = vector::length(v);
    while (i < n) {
        if (vector::borrow(v, i) == target) return option::some(i);
        i = i + 1
    };

    option::none()
}
```

不带参数使用`return`是`return ()`的简写。也就是说,以下两个函数是等价的:

```move
fun foo() { return }
fun foo() { return () }
```
