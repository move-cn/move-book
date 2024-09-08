# 宏函数

宏函数是一种在每次调用时在编译期间展开的函数定义方式。宏的参数不像普通函数那样被急切求值，而是通过表
达式进行替换。此外，调用者可以通过 [lambda](#lambdas) 向宏提供代码。

这种表达式替换机制使得 `macro` 函数类似于
[其他编程语言中的宏](<https://en.wikipedia.org/wiki/Macro_(computer_science)>); 然而，它们在 Move 中
比你从其他语言中预期的要更受限制。`macro` 函数的参数和返回值仍然是有类型的——尽管这可以部分放宽，通过
[`_` 类型](../generics.md#_-type)。然而，这一限制的好处在于，`macro` 函数可以在任何正常函数可以使用
的地方使用，这对于 [方法语法](../method-syntax.md) 特别有帮助。

未来可能会出现一个更广泛的
[语法宏](<https://en.wikipedia.org/wiki/Macro_(computer_science)#Syntactic_macros>) 系统。

## 语法

`macro` 函数的语法与普通函数类似。然而，所有类型参数名称和所有参数名称必须以
`translate from $sourceLang to $targetLang:` 开头。请注意，`_` 仍然可以单独使用，但不能作为前缀，而
必须使用 `$_`。

```text
<visibility>? macro fun <identifier><[$type_parameters: constraint],*>([$identifier: type],*): <return_type> <function_body>
```

例如，以下 `macro` 函数接受一个向量和一个 lambda，并将该 lambda 应用到向量的每个元素上，以构造一个新
的向量。

```move
macro fun map<$T, $U>($v: vector<$T>, $f: |$T| -> $U): vector<$U> {
    let mut v = $v;
    v.reverse();
    let mut i = 0;
    let mut result = vector[];
    while (!v.is_empty()) {
        result.push_back($f(v.pop_back()));
        i = i + 1;
    };
    result
}
```

`$` 是用来指示参数（类型参数和数值参数）与它们正常的非宏对应物行为不同。对于类型参数，它们可以用任何
类型实例化（甚至是引用类型 `&` 或 `&mut`），并且会满足任何约束。同样，对于参数，它们不会被急切求值，
而是在每次使用时替换为参数表达式。

## Lambda

Lambda 是一种新类型的表达式，仅可与 `macro` 一起使用。这些用于将代码从调用者传递到 `macro` 的主体中
。虽然替换是在编译时完成的，但它们的用法类似于其他语言中的
[匿名函数](https://en.wikipedia.org/wiki/Anonymous_function)、[lambda](https://en.wikipedia.org/wiki/Lambda_calculus)
或 [闭包](<https://en.wikipedia.org/wiki/Closure_(computer_programming)>).

如上例所示（`$f: |$T| -> $U`），lambda 类型的定义语法为

```text
|<type>,*| (-> <type>)?
```

一些例子：

```move
|u64, u64| -> u128 // 一个接受两个 u64 并返回一个 u128 的 lambda
|&mut vector<u8>| -> &mut u8 // 一个接受 &mut vector<u8> 并返回 &mut u8 的 lambda
```

如果返回类型未注解，则默认为 `()`。

```move
// 以下是等价的
|&mut vector<u8>, u64|
|&mut vector<u8>, u64| -> ()
```

Lambda 表达式在 `macro` 的调用站点处定义，语法如下：

```text
|(<identifier> (: <type>)?),*| <expression>
|(<identifier> (: <type>)?),*| -> <type> { <expression> }
```

注意，如果返回类型已注解，则 lambda 的主体必须用 `{}` 括起来。

使用上面定义的 `map` 宏，我们可以这样使用 lambda：

```move
let v = vector[1, 2, 3];
let doubled: vector<u64> = map!(v, |x| 2 * x);
let bytes: vector<vector<u8>> = map!(v, |x| std::bcs::to_bytes(&x));
```

并带有类型注解：

```move
let doubled: vector<u64> = map!(v, |x: u64| 2 * x); // 返回类型注解可选
let bytes: vector<vector<u8>> = map!(v, |x: u64| -> vector<u8> { std::bcs::to_bytes(&x) });
```

### 捕获

Lambda 表达式还可以引用在定义 lambda 的范围内声明的变量。这有时被称为 "捕获"。

```move
let res = foo();
let incremented = map!(vector[1, 2, 3], |x| x + res);
```

任何变量都可以被捕获，包括可变和不可变的变量。

请参见[示例](#iterating-over-a-vector)部分以获取更复杂的用法。

### 限制

目前，lambda 只能直接在 `macro` 调用中使用。它们不能绑定到变量。例如，以下代码将产生错误：

````move
let f = |x| 2 * x;
//      ^^^^^^^^^ Error! Lambdas must be used directly in 'macro' calls
let doubled: vector<u64> = map!(vector[1, 2, 3], f);

## 输入

与普通函数一样，`macro` 函数是有类型的——参数和返回值的类型必须被注解。然而，函数的主体在宏展开之前不会进行类型检查。这意味着给定宏的并非所有用法都是有效的。例如

```move
macro fun add_one<$T>($x: $T): $T {
    $x + 1
}
````

上述宏在`$T`不是原始整数类型时将不会进行类型检查。

这在与[方法语法](../method-syntax.md)结合使用时特别有用，因为函数直到宏展开后才会被解析。

```move
macro fun call_foo<$T, $U>($x: $T): &$U {
    $x.foo()
}
```

此宏只有在 `$T` 具有一个返回引用 `&$U` 的方法 `foo` 时才能成功展开。

如 [卫生](#hygiene) 部分所述，`foo` 将根据定义 `call_foo` 的作用域进行解析，而不是根据其展开的位置。

### 类型参数

类型参数可以用任何类型实例化，包括引用类型 `&` 和 `&mut`。它们

也可以用 [元组类型](../primitive-types/tuple.md) 实例化，尽管目前这的实用性有限，因为元组不能绑定到
变量上。

这种放宽要求迫使在调用点满足类型参数的约束，这种情况通常不会发生。然而，通常建议将所有必要的约束添加
到类型参数中。例如

```move
public struct NoAbilities()
public struct CopyBox<T: copy> has copy, drop { value: T }
macro fun make_box<$T>($x: $T): CopyBox<$T> {
    CopyBox { value: $x }
}
```

此宏仅在 `$T` 被实例化为具有 `copy` 能力的类型时扩展。

```move
make_box!(1); // Valid!
make_box!(NoAbilities()); // Error! 'NoAbilities' does not have the copy ability
```

建议的 `make_box` 声明是将 `copy` 约束添加到类型参数中。

这就向调用者传达了该类型必须具有 `copy` 能力。

```move
macro fun make_box<$T: copy>($x: $T): CopyBox<$T> {
    CopyBox { value: $x }
}
```

那么，人们可能会合理地问，既然建议不使用它，为什么还要放宽这个限制？类型参数的约束在所有情况下都无法
强制执行，因为函数体在展开之前不会被检查。在下面的示例中，`$T`上的`copy`约束在签名中不是必需的，但在
函数体中是必要的。

```move
macro fun read_ref<$T>($r: &$T): $T {
    *$r
}
```

然而，如果您想要一个极其宽松的类型签名，建议使用 [`_` 类型](#_-type)。

### `_` 类型

通常，[`_` 占位符类型](../generics.md#_-type) 在表达式中用于允许部分注解类型参数。然而，在 `macro`
函数中，可以使用 `_` 类型代替类型参数，以放宽任何类型的签名。这应该会提高声明“泛型” `macro` 函数的便
利性。

例如，我们可以

```move
macro fun add($x: _, $y: _, $z: _): u256 {
    ($x as u256) + ($y as u256) + ($z as u256)
}
```

此外，`_` 类型可以用不同的类型实例化多次。例如

```move
public struct Box<T> has copy, drop, store { value: T }
macro fun create_two($f: |_| -> Box<_>): (Box<u8>, Box<u16>) {
    ($f(0u8), $f(0u16))
}
```

如果我们改为使用类型参数声明函数，那么这些类型必须统一为一个共同的类型，而在这种情况下这是不可能的。

```move
macro fun create_two<$T>($f: |$T| -> Box<$T>): (Box<u8>, Box<u16>) {
    ($f(0u8), $f(0u16))
    //           ^^^^ Error! expected `u8` but found `u16`
}
...
let (a, b) = create_two!(|value| Box { value });
```

在这种情况下，`$T` 必须被实例化为单一类型，但推断发现 `$T` 必须同时绑定到 `u8` 和 `u16`。

然而，这存在一个权衡，因为 `_` 类型对调用者传达的意义和意图较少。

考虑将上面重新声明的 `map` 宏用 `_` 替代 `$T` 和 `$U`。

```move
macro fun map($v: vector<_>, $f: |_| -> _): vector<_> {
```

在类型层面上不再有关于 `$f` 行为的任何指示。调用者必须从注释或宏的主体中获取理解。

## 扩展和替换

`macro` 的主体在编译时被替换到调用位置。每个参数都由其实参的 _表达式_ 替代，而不是值。对于 lambda，
额外的局部变量可以在 `macro` 主体的上下文中绑定值。

举一个非常简单的例子

```move
macro fun apply($f: |u64| -> u64, $x: u64): u64 {
    $f($x)
}
```

在调用站点

```move
let incremented = apply!(|x| x + 1, 5);
```

这将会被扩展为

```move
let incremented = {
    let x = { 5 };
    { x + 1 }
};
```

再次，`x` 的值不会被替换，而是表达式 `5` 被替换。这可能意味着根据 `macro` 的主体，参数可能被多次评估
，

```move
macro fun dup($f: |u64, u64| -> u64, $x: u64): u64 {
    $f($x, $x)
}
```

```move
let sum = dup!(|x, y| x + y, foo());
```

被扩展为

```move
let sum = {
    let x = { foo() };
    let y = { foo() };
    { x + y }
};
```

请注意，`foo()` 将被调用两次。如果 `dup` 是一个普通函数，这种情况是不会发生的。

通常建议通过将参数绑定到局部变量来创建可预测的评估行为。

```move
macro fun dup($f: |u64, u64| -> u64, $x: u64): u64 {
    let a = $x;
    $f(a, a)
}
```

现在同一个调用位置将扩展为

```move
let sum = {
    let a = { foo() };
    {
        let x = { a };
        let y = { a };
        { x + y }
    }
};
```

### 卫生

在上面的例子中，`dup` 宏有一个局部变量 `a`，用于绑定参数 `$x`。你可能会问，如果这个变量被命名为 `x`
会发生什么？这会与 lambda 中的 `x` 冲突吗？

简短的回答是，不会。`macro` 函数是 [卫生的](https://en.wikipedia.org/wiki/Hygienic_macro)，这意味着
宏和 lambda 的展开不会意外捕获来自其他作用域的变量。

编译器通过将每个作用域关联一个唯一编号来实现这一点。当 `macro` 被展开时，宏体获得自己的作用域。此外
，每次使用时参数都会重新定义作用域。

修改 `dup` 宏以使用 `x` 而不是 `a`.

```move
macro fun dup($f: |u64, u64| -> u64, $x: u64): u64 {
    let a = $x;
    $f(a, a)
}
```

调用位置的扩展

```move
// let sum = dup!(|x, y| x + y, foo());
let sum = {
    let x#1 = { foo() };
    {
        let x#2 = { x#1 };
        let y#2 = { x#1 };
        { x#2 + y#2 }
    }
};
```

这是编译器内部表示的一个近似，某些细节为了简化这个例子而被省略。

每次使用参数时都会重新作用域，以确保不同的用法不会冲突。

```move
macro fun apply_twice($f: |u64| -> u64, $x: u64): u64 {
    $f($x) + $f($x)
}
```

```move
let result = apply_twice!(|x| x + 1, { let x = 5; x });
```

展开为

```move
let result = {
    {
        let x#1 = { let x#2 = { 5 }; x#2 };
        { x#1 + x#1 }
    }
    +
    {
        let x#3 = { let x#4 = { 5 }; x#4 };
        { x#3 + x#3 }
    }
};
```

类似地，[方法解析](../method-syntax.md) 也被限制在宏定义的范围内。例如

```move
public struct S { f: u64, g: u64 }

fun f(s: &S): u64 {
    s.f
}
fun g(s: &S): u64 {
    s.g
}

use fun f as foo;
macro fun call_foo($s: &S): u64 {
    let s = $s;
    s.foo()
}
```

在这种情况下，方法调用 `foo` 始终会解析为函数 `f`，即使 `call_foo` 在一个 `foo` 绑定到不同函数（例如
`g`）的作用域中使用。

```move
fun example(s: &S): u64 {
    use fun g as foo;
    call_foo!(s) // expands to 'f(s)', not 'g(s)'
}
```

由于这个原因，未使用的 `use fun` 声明在包含 `macro` 函数的模块中可能不会收到警告。

### 控制流

与变量卫生类似，控制流构造也始终作用于它们被定义的位置，而不是它们被扩展的位置。

```move
macro fun maybe_div($x: u64, $y: u64): u64 {
    let x = $x;
    let y = $y;
    if (y == 0) return 0;
    x / y
}
```

在调用站点，`return` 将始终从 `macro` 主体返回，而不是从调用者返回。

```move
let result: vector<u64> = vector[maybe_div!(10, 0)];
```

将扩展为

```move
let result: vector<u64> = vector['a: {
    let x = { 10 };
    let y = { 0 };
    if (y == 0) return 'a 0;
    x / y
}];
```

其中 `return 'a 0` 将返回到块 `'a: { ... }` 而不是调用者的主体。有关更多详细信息，请参见
[带标签的控制流](../control-flow/labeled-control-flow.md) 部分。

同样，lambda 中的 `return` 将从 lambda 返回，而不是从 `macro` 主体或外部函数返回。

```move
macro fun apply($f: |u64| -> u64, $x: u64): u64 {
    $f($x)
}
```

和

```move
let result = apply!(|x| { if (x == 0) return 0; x + 1 }, 100);
```

将扩展为

```move
let result = {
    let x = { 100 };
    'a: {
        if (x == 0) return 'a 0;
        x + 1
    }
};
```

除了从 lambda 返回外，标签还可以用于返回到外部函数。在 `vector::any` 宏中，使用带标签的 `return` 可
以提前从整个 `macro` 返回。

```move
public macro fun any<$T>($v: &vector<$T>, $f: |&$T| -> bool): bool {
    let v = $v;
    'any: {
        v.do_ref!(|e| if ($f(e)) return 'any true);
        false
    }
}
```

`return 'any true` 在条件满足时提前退出“循环”。否则，宏将“返回” `false`。

### 方法语法

在适用的情况下，可以使用[方法语法](../method-syntax.md)调用`宏`函数。当使用方法语法时，参数的求值方
式将发生变化，第一个参数（方法的“接收者”）将在宏扩展之外进行求值。这个例子虽然有些牵强，但可以简明地
展示这种行为:

```move
public struct S() has copy, drop;
public fun foo(): S { abort 0 }
public macro fun maybe_s($s: S, $cond: bool): S {
    if ($cond) $s
    else S()
}
```

即使 `foo()` 会中止，它的返回类型仍然可以用于开始一个方法调用。

如果 `$cond` 为 `false`，则 `$s` 不会被评估，并且在正常的非方法调用下，参数

`foo()` 将不会被评估，也不会中止。以下示例演示了在参数为 `foo()` 时 `$s` 没有被评估。

```move
maybe_s!(foo(), false) // does not abort
```

当查看扩展形式时，为什么它不中止变得更加清晰。

```move
if (false) foo()
else S()
```

然而，当使用方法语法时，第一个参数在宏展开之前被评估。因此，`foo()` 的相同参数 `$s` 现在将被评估并会
中止。

```move
foo().maybe_s!(false) // aborts
```

当我们查看展开形式时，这一点会更加清晰。

```move
let tmp = foo(); // aborts
if (false) tmp
else S()
```

从概念上讲，方法调用的接收者在宏展开之前被绑定到一个临时变量，这迫使评估，从而导致中止。

### 参数限制

`macro` 函数的参数必须始终作为表达式使用。它们不能在可能被重新解释的情况下使用。例如，以下情况是不允
许的

```move
macro fun no($x: _): _ {
    $x.f
}
```

原因是，如果参数 `$x` 不是引用，它将首先被借用，这可能会重新解释该参数。为了绕过这个限制，您应该将参
数绑定到一个局部变量。

```move
macro fun yes($x: _): _ {
    let x = $x;
    x.f
}
```

## 示例

### 懒惰参数：assert_eq

```move
macro fun assert_eq<$T>($left: $T, $right: $T, $code: u64) {
    let left = $left;
    let right = $right;
    if (left != right) {
        std::debug::print(&b"assertion failed.\n left: ");
        std::debug::print(&left);
        std::debug::print(&b"\n does not equal right: ");
        std::debug::print(&right);
        abort $code;
    }
}
```

在这种情况下，只有当断言失败时，`$code` 的参数才会被评估。

```move
assert_eq!(vector[true, false], vector[true, false], 1 / 0); // division by zero is not evaluated
```

### 任意整数平方根

此宏计算任何整数类型的整数平方根，除了 `u256`。

`$T` 是输入的类型，`$bitsize` 是该类型中的位数，例如 `u8` 有 8 位。 `$U` 应设置为下一个更大的整数类
型，例如对于 `u8` 使用 `u16`。

在这个 `macro` 中，整数字面量的类型是被注释的，比如 `(1: $U)`，允许每次调用时字面量的类型不同。同样
，可以使用关键字 `as` 与类型参数 `$T` 和 `$U`. 只有当 `$T` 和 `$U$ 被实例化为整数类型时，此宏才会成
功扩展。

```move
macro fun num_sqrt<$T, $U>($x: $T, $bitsize: u8): $T {
    let x = $x;
    let mut bit = (1: $U) << $bitsize;
    let mut res = (0: $U);
    let mut x = x as $U;

    while (bit != 0) {
        if (x >= res + bit) {
            x = x - (res + bit);
            res = (res >> 1) + bit;
        } else {
            res = res >> 1;
        };
        bit = bit >> 2;
    };

    res as $T
}
```

### 遍历向量

这两个 `宏` 分别以不可变和可变的方式遍历一个向量。

```move
macro fun for_imm<$T>($v: &vector<$T>, $f: |&$T|) {
    let v = $v;
    let n = v.length();
    let mut i = 0;
    while (i < n) {
        $f(&v[i]);
        i = i + 1;
    }
}

macro fun for_mut<$T>($v: &mut vector<$T>, $f: |&mut $T|) {
    let v = $v;
    let n = v.length();
    let mut i = 0;
    while (i < n) {
        $f(&mut v[i]);
        i = i + 1;
    }
}
```

一些使用示例:

```move
fun imm_examples(v: &vector<u64>) {
    // print all elements
    for_imm!(v, |x| std::debug::print(x));

    // sum all elements
    let mut sum = 0;
    for_imm!(v, |x| sum = sum + x);

    // find the max element
    let mut max = 0;
    for_imm!(v, |x| if (x > max) max = x);
}

fun mut_examples(v: &mut vector<u64>) {
    // increment each element
    for_mut!(v, |x| *x = *x + 1);

    // set each element to the previous value, and the first to last value
    let mut prev = v[v.length() - 1];
    for_mut!(v, |x| {
        let tmp = *x;
        *x = prev;
        prev = tmp;
    });

    // set the max element to 0
    let mut max = &mut 0;
    for_mut!(v, |x| if (*x > *max) max = x);
    *max = 0;
}
```

### 非循环 lambda 使用

Lambda 不需要在循环中使用，通常用于有条件地应用代码。

```move
macro fun inspect<$T>($opt: &Option<$T>, $f: |&$T|) {
    let opt = $opt;
    if (opt.is_some()) $f(opt.borrow())
}

macro fun is_some_and<$T>($opt: &Option<$T>, $f: |&$T| -> bool): bool {
    let opt = $opt;
    if (opt.is_some()) $f(opt.borrow())
    else false
}

macro fun map<$T, $U>($opt: Option<$T>, $f: |$T| -> $U): Option<$U> {
    let opt = $opt;
    if (opt.is_some()) {
        option::some($f(opt.destroy_some()))
    } else {
        opt.destroy_none();
        option::none()
    }
}
```

以及一些用法示例:

```move
fun examples(opt: Option<u64>) {
    // print the value if it exists
    inspect!(&opt, |x| std::debug::print(x));

    // check if the value is 0
    let is_zero = is_some_and!(&opt, |x| *x == 0);

    // upcast the u64 to a u256
    let str_opt = map!(opt, |x| x as u256);
}
```
