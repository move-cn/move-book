# Local Variables and Scope(局部变量和作用域)

在 Move 中，局部变量是词法（静态）作用域的。新变量通过关键字 `let` 引入，这将遮蔽任何具有相同名称的先前局部变量。标记为 `mut` 的局部变量是可变的，可以直接修改或通过可变引用进行修改。

## 声明局部变量

### `let` 绑定

Move 程序使用 `let` 将变量名称绑定到值：

```move
let x = 1;
let y = x + x;
```

`let` 也可以在不将值绑定到局部变量的情况下使用。

```move
let x;
```

然后可以在稍后为局部变量赋值。

```move
let x;
if (cond) {
  x = 1;
} else {
  x = 0;
}
```

在无法提供默认值时，这在从循环中提取值时非常有用。

```move
let x;
let cond = true;
let i = 0;
loop {
    let (res, cond) = foo(i);
    if (!cond) {
        x = res;
        break;
    };
    i = i + 1;
}
```

要在赋值后修改局部变量，或者借用它的可变引用 `&mut`，必须将其声明为 `mut`。

```move
let mut x = 0;
if (cond) x = x + 1;
foo(&mut x);
```

有关更多详细信息，请参见下面的 [赋值](#assignments) 部分。

### 变量必须在使用前赋值

Move 的类型系统防止局部变量在赋值前使用。

```move
let x;
x + x // 错误！x 在赋值前被使用
```

```move
let x;
if (cond) x = 0;
x + x // 错误！x 并不是在所有情况下都有值
```

```move
let x;
while (cond) x = 0;
x + x // 错误！x 并不是在所有情况下都有值
```

### 有效的变量名

变量名可以包含下划线 `_`、字母 `a` 到 `z`、字母 `A` 到 `Z` 和数字 `0` 到 `9`。变量名必须以下划线 `_` 或字母 `a` 到 `z` 开头。它们不能以大写字母开头。

```move
// 所有合法的变量名
let x = e;
let _x = e;
let _A = e;
let x0 = e;
let xA = e;
let foobar_123 = e;

// 所有非法的变量名
let X = e; // 错误！
let Foo = e; // 错误！
```

### 类型注解

局部变量的类型几乎总是可以通过 Move 的类型系统推断出来。然而，Move 允许显式的类型注解，这对于可读性、清晰性或调试非常有用。添加类型注解的语法是：

```move
let x: T = e; // "变量 x 的类型为 T，被初始化为表达式 e"
```

一些显式类型注解的例子：

```move
module 0x42::example {

    public struct S { f: u64, g: u64 }

    fun annotated() {
        let u: u8 = 0;
        let b: vector<u8> = b"hello";
        let a: address = @0x0;
        let (x, y): (&u64, &mut u64) = (&0, &mut 1);
        let S { f, g: f2 }: S = S { f: 0, g: 1 };
    }
}
```

注意，类型注解必须始终位于模式的右侧：

```move
// 错误！应为 let (x, y): (&u64, &mut u64) = ...
let (x: &u64, y: &mut u64) = (&0, &mut 1);
```

### 何时需要注解

在某些情况下，如果类型系统无法推断类型，则需要局部类型注解。这通常发生在无法推断泛型类型的类型参数时。例如：

```move
let _v1 = vector[]; // 错误！
//        ^^^^^^^^ 无法推断此类型。尝试添加注解
let v2: vector<u64> = vector[]; // 无错误
```

在较少见的情况下，如果类型系统无法推断出分歧代码（所有后续代码都不可达）的类型，也可能需要类型注解。`return` 和 `abort` 都是表达式，可以具有任何类型。如果一个 `loop` 具有 `break`，则其类型为 `()`（或者如果具有 `break e` 且 `e: T`，则类型为 `T`），但如果没有跳出 `loop`，它可以具有任何类型。如果这些类型无法推断，则需要类型注解。例如，此代码：

```move
let a: u8 = return ();
let b: bool = abort 0;
let c: signer = loop ();

let x = return (); // 错误！
//  ^ 无法推断此类型。尝试添加注解
let y = abort 0; // 错误！
//  ^ 无法推断此类型。尝试添加注解
let z = loop (); // 错误！
//  ^ 无法推断此类型。尝试添加注解
```

为此代码添加类型注解将会暴露其他关于无效代码或未使用的局部变量的错误，但这个例子对于理解这个问题仍然有帮助。

### 使用元组的多个声明

`let` 可以使用元组一次引入多个局部变量。括号内声明的局部变量初始化为元组中的相应值。

```move
let () = ();
let (x0, x1) = (0, 1);
let (y0, y1, y2) = (0, 1, 2);
let (z0, z1, z2, z3) = (0, 1, 2, 3);
```

表达式的类型必须完全匹配元组模式的元数。

```move
let (x, y) = (0, 1, 2); // 错误！
let (x, y, z, q) = (0, 1, 2); // 错误！
```

不能在单个 `let` 中声明多个具有相同名称的局部变量。

```move
let (x, x) = 0; // 错误！
```

声明的局部变量的可变性可以混合。

```move
let (mut x, y) = (0, 1);
x = 1;
```

### 使用结构体的多个声明

`let` 还可以在解构（或匹配）结构体时一次引入多个局部变量。在这种形式中，`let` 创建一组局部变量，这些变量被初始化为结构体字段的值。语法如下所示：

```move
public struct T { f1: u64, f2: u64 }
```

```move
let T { f1: local1, f2: local2 } = T { f1: 1, f2: 2 };
// local1: u64
// local2: u64
```

类似地，对于位置结构体：

```move
public struct P(u64, u64)
```

和

```move
let P (local1, local2) = P (1, 2);
// local1: u64
// local2: u64
```

下面是一个更复杂的例子：

```move
module 0x42::example {
    public struct X(u64);
    public struct Y { x1: X, x2: X }

    fun new_x(): X {
        X(1)
    }

    fun example() {
        let Y { x1: X(f), x2 } = Y { x1: new_x(), x2: new_x() };
        assert!(f + x2.0 == 2, 42);

        let Y { x1: X(f1), x2: X(f2) } = Y { x1: new_x(), x2: new_x() };
        assert!(f1 + f2 == 2, 42);
    }
}
```

结构体的字段可以起到双重作用，既标识要绑定的字段，又标识变量的名称。这有时被称为捣鬼（punning）。

```move
let Y { x1, x2 } = e;
```

等价于：

```move
let Y { x1: x1, x2: x2 } = e;
```

如元组所示，不能在单个 `let` 中声明多个具有相同名称的局部变量。

```move
let Y { x1: x, x2: x } = e; // 错误！
```

同样地，声明的局部变量的可变性可以混合。

```move
let Y { x1: mut x1, x2 } = e;
```

此外，可变性注解可以应用于捣鬼字段。给出等价的例子：

```move
let Y { mut x1, x2 } = e;
```

### 针对引用的解构

在上述结构体的例子中，`let` 中绑定的值被移动，销毁了结构体值并绑定其字段。

```move
public struct T { f1: u64, f2: u64 }
```

```move
let T { f1: local1, f2: local2 } = T { f1: 1, f2: 2 };
// local1: u64
// local2: u64
```

在这种情况下，结构体值 `T { f1: 1, f2: 2 }` 在 `let` 之后不再存在。

如果您希望不移动和销毁结构体值，可以借用它的每个字段。例如：

```move
let t = T { f1: 1, f2: 2 };
let T { f1: local1, f2: local2 } = &t;
// local1: &u64
// local2: &u64
```

同样地，对于可变引用：

```move
let mut t = T { f1: 1, f2: 2 };
let T { f1: local1, f2: local2 } = &mut t;
// local1: &mut u64
// local2: &mut u64
```

这种行为也适用于嵌套结构体。

```move
module 0x42::example {
    public struct X(u64);
    public struct Y { x1: X, x2: X }

    fun new_x(): X {
        X(1)
    }

    fun example() {
        let mut y = Y { x1: new_x(), x2: new_x() };

        let Y { x1: X(f), x2 } = &y;
        assert!(*f + x2.0 == 2, 42);

        let Y { x1: X(f1), x2: X(f2) } = &mut y;
        *f1 = *f1 + 1;
        *f2 = *f2 + 1;
        assert!(*f1 + *f2 == 4, 42);
    }
}
```

### 忽略数值

在 `let` 绑定中，有时候忽略一些数值是很有帮助的。以 `_` 开头的局部变量会被忽略，不会引入新的变量。

```move
fun three(): (u64, u64, u64) {
    (0, 1, 2)
}
```

```move
let (x1, _, z1) = three();
let (x2, _y, z2) = three();
assert!(x1 + z1 == x2 + z2, 42);
```

有时候这是必要的，因为编译器会对未使用的局部变量发出警告。

```move
let (x1, y, z1) = three(); // 警告！
//       ^ 未使用的局部变量 'y'
```

### 通用的 `let` 语法

所有 `let` 语句中的不同结构都可以结合在一起！由此得出了 `let` 语句的通用语法：

> _let-binding_ → **let** _pattern-or-list_ _type-annotation_<sub>_opt_</sub> _initializer_<sub>_opt_</sub>  
> _pattern-or-list_ → _pattern_ | **(** _pattern-list_ **)**  
> _pattern-list_ → _pattern_ **,**<sub>_opt_</sub> | _pattern_ **,** _pattern-list_  
> _type-annotation_ → **:** _type_  
> _initializer_ → **=** _expression_

引入绑定的项目的通用术语是 _pattern_。模式既用于解构数据（可能是递归的），也用于引入绑定。模式的语法如下：

> _pattern_ → _local-variable_ | _struct-type_ **{** _field-binding-list_ **}**  
> _field-binding-list_ → _field-binding_ **,**<sub>_opt_</sub> | _field-binding_ **,** _field-binding-list_  
> _field-binding_ → _field_ | _field_ **:** _pattern_

使用此语法的几个具体示例：

```move
    let (x, y): (u64, u64) = (0, 1);
//       ^                           local-variable
//       ^                           pattern
//          ^                        local-variable
//          ^                        pattern
//          ^                        pattern-list
//       ^^^^                        pattern-list
//      ^^^^^^                       pattern-or-list
//            ^^^^^^^^^^^^           type-annotation
//                         ^^^^^^^^  initializer
//  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ let-binding

    let Foo { f, g: x } = Foo { f: 0, g: 1 };
//      ^^^                                    struct-type
//            ^                                field
//            ^                                field-binding
//               ^                             field
//                  ^                          local-variable
//                  ^                          pattern
//               ^^^^                          field-binding
//            ^^^^^^^                          field-binding-list
//      ^^^^^^^^^^^^^^^                        pattern
//      ^^^^^^^^^^^^^^^                        pattern-or-list
//                      ^^^^^^^^^^^^^^^^^^^^   initializer
//  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ let-binding
```

## 变更

### 赋值

在引入局部变量后（通过 `let` 或作为函数参数），可以通过赋值修改 `mut` 局部变量：

```move
x = e
```

与 `let` 绑定不同，赋值是表达式。在某些语言中，赋值返回被赋的值，但在 Move 中，任何赋值的类型始终为 `()`。

```move
(x = e: ())
```

实际上，赋值作为表达式意味着它们可以在不使用大括号（`{`...`}`）添加新表达式块的情况下使用。

```move
let x;
if (cond) x = 1 else x = 2;
```

赋值使用与 `let` 绑定类似的模式语法方案，但不包含 `mut`：

```move
module 0x42::example {
    public struct X { f: u64 }

    fun new_x(): X {
        X { f: 1 }
    }

    // 注意：此示例会有关于未使用变量和赋值的警告。
    fun example() {
       let (mut x, mut y, mut f, mut g) = (0, 0, 0, 0);

       (X { f }, X { f: x }) = (new_x(), new_x());
       assert!(f + x == 2, 42);

       (x, y, f, _, g) = (0, 0, 0, 0, 0);
    }
}
```

注意，局部变量只能有一种类型，因此在赋值之间不能改变局部变量的类型。

```move
let mut x;
x = 0;
x = false; // 错误！
```

### 通过引用进行修改

除了直接使用赋值修改局部变量外，还可以通过可变引用 `&mut` 修改 `mut` 局部变量。

```move
let mut x = 0;
let r = &mut x;
*r = 1;
assert!(x == 1, 42);
```

这特别有用，如果：

(1) 你想根据某些条件修改不同的变量。

```move
let mut x = 0;
let mut y = 1;
let r = if (cond) &mut x else &mut y;
*r = *r + 1;
```

(2) 你希望另一个函数修改你的局部值。

```move
let mut x = 0;
modify_ref(&mut x);
```

这种修改方式也适用于修改结构体和向量！

```move
let mut v = vector[];
vector::push_back(&mut v, 100);
assert!(*vector::borrow(&v, 0) == 100, 42);
```

更多细节请参阅 [Move references](./primitive-types/references.md)。

## 作用域

任何用 `let` 声明的局部变量，在其后的任何表达式中均可用，在该作用域内有效。作用域由表达式块 `{`...`}` 声明。

局部变量不能在声明的作用域之外使用。

```move
let x = 0;
{
    let y = 1;
};
x + y // 错误！
//  ^ 未绑定的局部变量 'y'
```

但是，外部作用域的局部变量可以在嵌套作用域中使用。

```move
{
    let x = 0;
    {
        let y = x + 1; // 合法
    }
}
```

局部变量可以在其可访问的任何作用域中进行修改。该变更将随着局部变量一起生存，不论执行变更的作用域如何。

```move
let mut x = 0;
x = x + 1;
assert!(x == 1, 42);
{
    x = x + 1;
    assert!(x == 2, 42);
};
assert!(x == 2, 42);
```

### 表达式块

表达式块是一系列由分号 (`;`) 分隔的语句。表达式块的结果值是块中最后一个表达式的值。

```move
{ let x = 1; let y = 1; x + y }
```

在这个例子中，块的结果是 `x + y` 的值。

语句可以是 `let` 声明或表达式。请记住，赋值语句 (`x = e`) 是类型为 `()` 的表达式。

```move
{ let x; let y = 1; x = 1; x + y }
```

函数调用是另一种常见的类型为 `()` 的表达式。修改数据的函数调用通常用作语句。

```move
{ let v = vector[]; vector::push_back(&mut v, 1); v }
```

这不仅限于 `()` 类型 --- 任何表达式都可以作为序列中的语句使用！

```move
{
    let x = 0;
    x + 1; // 值被丢弃
    x + 2; // 值被丢弃
    b"hello"; // 值被丢弃
}
```

但是！如果表达式中包含资源（即没有 `drop` [能力](./abilities.md) 的值），你会收到错误消息。这是因为 Move 的类型系统保证任何被丢弃的值都具有 `drop` [能力](./abilities.md)。（所有权必须在声明模块内部转移或显式销毁该值。）

```move
{
    let x = 0;
    Coin { value: x }; // 错误！
//  ^^^^^^^^^^^^^^^^^ 未使用值且缺少 `drop` 能力
    x
}
```

如果在块中没有最终表达式 --- 也就是说，如果有一个尾随的分号 `;`，那么会有一个隐式的 [单元 `()` 值](https://en.wikipedia.org/wiki/Unit_type)。同样地，如果表达式块为空，也有一个隐式的单元 `()` 值。

两者是等效的

```move
{ x = x + 1; 1 / x; }
```

```move
{ x = x + 1; 1 / x; () }
```

同样地，这两者也是等效的

```move
{ }
```

```move
{ () }
```

表达式块本身是一个表达式，并且可以在任何需要表达式的地方使用。（注意：函数体本身是一个表达式块，但函数体不能被其他表达式替换。）

```move
let my_vector: vector<vector<u8>> = {
    let mut v = vector[];
    vector::push_back(&mut v, b"hello");
    vector::push_back(&mut v, b"goodbye");
    v
};
```

（在这个例子中不需要类型注释，只是为了清晰起见添加的。）

### 遮蔽

如果 `let` 引入的局部变量与作用域中已有的变量同名，那么之前的变量在此作用域后将无法访问。这称为 _遮蔽_。

```move
let x = 0;
assert!(x == 0, 42);

let x = 1; // x 被遮蔽
assert!(x == 1, 42);
```

当一个局部变量被遮蔽时，它不需要保留之前的类型。

```move
let x = 0;
assert!(x == 0, 42);

let x = b"hello"; // x 被遮蔽
assert!(x == b"hello", 42);
```

局部变量被遮蔽后，其值仍然存在，但将不再可访问。这点在处理没有 [`drop` 能力](./abilities.md) 的类型的值时尤为重要，因为该值的所有权必须在函数结束前转移。

```move
module 0x42::example {
    public struct Coin has store { value: u64 }

    fun unused_coin(): Coin {
        let x = Coin { value: 0 }; // 错误！
//          ^ 此局部变量仍包含没有 `drop` 能力的值
        x.value = 1;
        let x = Coin { value: 10 };
        x
//      ^ 返回无效
    }
}
```

当局部变量在作用域内被遮蔽时，遮蔽仅在该作用域内有效。一旦作用域结束，遮蔽就消失了。

```move
let x = 0;
{
    let x = 1;
    assert!(x == 1, 42);
};
assert!(x == 0, 42);
```

请记住，局部变量在被遮蔽时可以改变类型。

```move
let x = 0;
{
    let x = b"hello";
    assert!(x = b"hello", 42);
};
assert!(x == 0, 42);
```

## Move 和 Copy

在 Move 中，所有局部变量可以通过 `move` 或 `copy` 两种方式使用。如果没有明确指定其中一种，Move 编译器可以推断出应该使用 `copy` 还是 `move`。这意味着在上述所有例子中，编译器会插入 `move` 或 `copy`。

从其他编程语言过来的人会更熟悉 `copy`，因为它会创建变量内部值的新副本以供表达式使用。使用 `copy`，局部变量可以多次使用。

```move
let x = 0;
let y = copy x + 1;
let z = copy x + 2;
```

任何具有 `copy` [能力](./abilities.md) 的值都可以以此方式复制，并且除非指定了 `move`，否则会自动复制。

`move` 将值从局部变量中取出，而不复制数据。一旦发生 `move`，该局部变量将不再可用，即使值的类型具有 `copy`
[能力](./abilities.md) 也是如此。

```move
let x = 1;
let y = move x + 1;
//      ------ 局部变量在此处被移动
let z = move x + 2; // 错误！
//      ^^^^^^ 无效使用局部变量 'x'
y + z
```

### 安全性

Move 的类型系统将阻止在值移动后继续使用该值。这与[`let`声明](#let-bindings)中描述的安全检查相同，防止局部变量在分配值之前被使用。

<!-- 更多信息，请参见 TODO 有关所有权和移动语义的未来部分。 -->

### 推断

如上所述，如果未指定 `copy` 或 `move`，Move 编译器会推断出应该使用 `copy` 还是 `move`。该算法非常简单：

- 任何具有 `copy` [能力](./abilities.md) 的值被视为 `copy`。
- 任何引用（可变 `&mut` 和不可变 `&`）被视为 `copy`。
  - 除了在特殊情况下，为了可预测的借用检查错误而被视为 `move`。这将在引用不再使用时发生。
- 其他任何值被视为 `move`。

给定以下结构体

```move
public struct Foo has copy, drop, store { f: u64 }
public struct Coin has store { value: u64 }
```

我们有以下示例

```move
let s = b"hello";
let foo = Foo { f: 0 };
let coin = Coin { value: 0 };
let coins = vector[Coin { value: 0 }, Coin { value: 0 }];

let s2 = s; // copy
let foo2 = foo; // copy
let coin2 = coin; // move
let coins2 = coin; // move

let x = 0;
let b = false;
let addr = @0x42;
let x_ref = &x;
let coin_ref = &mut coin2;

let x2 = x; // copy
let b2 = b; // copy
let addr2 = @0x42; // copy
let x_ref2 = x_ref; // copy
let coin_ref2 = coin_ref; // copy
```
