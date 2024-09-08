# Generics（泛型）

泛型可以用来定义函数和结构体，使它们可以适用于不同的输入数据类型。在 Move 中，我们通常将这种语言特性
称为参数化多态性。泛型参数和类型参数的术语在 Move 中可以互换使用。

泛型通常用于库代码中，例如在 [vector](./primitive-types/vector.md) 中，用于声明可以处理任何可能类型
（满足指定约束条件）的代码。这种参数化允许您在多种类型和情况下重用相同的实现。

## 声明类型参数

函数和结构体可以在它们的签名中使用一组类型参数列表，用尖括号 `<...>` 括起来。

### 泛型函数

函数的类型参数位于函数名之后和（值）参数列表之前。以下代码定义了一个泛型的身份函数，它接受任何类型的
值并返回该值本身。

```move
fun id<T>(x: T): T {
    // 这种类型注解是不必要的但是有效的
    (x: T)
}
```

一旦定义，类型参数 `T` 可以在参数类型、返回类型和函数体内部使用。

### 泛型结构体

结构体的类型参数位于结构体名字之后，可以用来命名字段的类型。

```move
public struct Foo<T> has copy, drop { x: T }

public struct Bar<T1, T2> has copy, drop {
    x: T1,
    y: vector<T2>,
}
```

请注意，[类型参数不一定要被使用](#unused-type-parameters)。

## 类型参数

### 调用泛型函数

在调用泛型函数时，可以在尖括号内指定函数的类型参数。

```move
fun foo() {
    let x = id<bool>(true);
}
```

如果您没有指定类型参数，Move 的 [类型推断](#type-inference) 将为您提供它们。

### 使用泛型结构体

类似地，可以在构造或销毁泛型类型的值时附加一个类型参数列表。

```move
fun foo() {
    // 构造时的类型参数
    let foo = Foo<bool> { x: true };
    let bar = Bar<u64, u8> { x: 0, y: vector<u8>[] };

    // 销毁时的类型参数
    let Foo<bool> { x } = foo;
    let Bar<u64, u8> { x, y } = bar;
}
```

在任何情况下，如果您没有指定类型参数，Move 的 [类型推断](#type-inference) 将为您提供它们。

### 类型参数不匹配

如果指定的类型参数与实际提供的值冲突，则会产生错误：

```move
fun foo() {
    let x = id<u64>(true); // 错误！true 不是 u64
}
```

类似地：

```move
fun foo() {
    let foo = Foo<bool> { x: 0 }; // 错误！0 不是 bool
    let Foo<address> { x } = foo; // 错误！bool 与 address 不兼容
}
```

## 类型推断

在大多数情况下，Move 编译器能够推断出类型参数，因此您不必显式地写出它们。如果省略类型参数，以下是上
面示例的代码：

```move
fun foo() {
    let x = id(true);
    //        ^ <bool> 被推断出来了

    let foo = Foo { x: true };
    //           ^ <bool> 被推断出来了

    let Foo { x } = foo;
    //     ^ <bool> 被推断出来了
}
```

注意：当编译器无法推断类型时，您需要手动注释它们。一个常见的场景是调用一个只在返回位置使用类型参数的
函数。

```move
module a::m {

    fun foo() {
        let v = vector[]; // 错误！
        //            ^ 编译器无法确定元素类型，因为它从未被使用过

        let v = vector<u64>[];
        //            ^~~~~ 在这种情况下必须手动注释
    }
}
```

请注意，这些情况有些刻意，因为 `vector[]` 从未被使用，因此 Move 的类型推断不能推断出类型。

然而，如果稍后在该函数中使用该值，编译器将能够推断出类型：

```move
module a::m {
    fun foo() {
        let v = vector[];
        //            ^ <u64> 被推断出来了
        vector::push_back(&mut v, 42);
        //               ^ <u64> 被推断出来了
    }
}
```

### `_` 类型

在某些情况下，您可能希望显式注释一些类型参数，但让编译器推断其他参数。`_` 类型作为编译器推断类型的占
位符。

```move
let bar = Bar<u64, _> { x: 0, y: vector[b"hello"] };
//                 ^ vector<u8> is inferred
```

占位符 `_` 只能出现在表达式和宏函数定义中，而不能出现在签名中。

这意味着您不能将 `_` 用作函数参数、函数返回类型、常量定义类型和数据字段的定义的一部分。

## 整数

在 Move 中，整数类型 `u8`、`u16`、`u32`、`u64`、`u128` 和 `u256` 都是不同的类型。然而，每种类型都可
以用相同的数值语法创建。换句话说，如果没有提供类型后缀，编译器将根据值的使用情况推断整数类型。

```move
let x8: u8 = 0;
let x16: u16 = 0;
let x32: u32 = 0;
let x64: u64 = 0;
let x128: u128 = 0;
let x256: u256 = 0;
```

如果值在不需要特定整数类型的上下文中未被使用，`u64` 将作为默认值。

```move
let x = 0;
//      ^ 默认使用 u64
```

然而，如果值对于推断类型太大，将会产生错误。

```move
let i: u8 = 256; // 错误！
//          ^^^ 对于 u8 来说太大了
let x = 340282366920938463463374607431768211454;
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ 对于 u64 来说太大了
```

在数字太大的情况下，您可能需要显式注释它：

```move
let x = 340282366920938463463374607431768211454u128;
//                                             ^^^^ 有效！
```

## 未使用的类型参数

对于结构体定义，未使用的类型参数是指在结构体定义的任何字段中都没有出现的类型参数，但在编译时会进行静
态检查。Move 允许未使用的类型参数，因此以下结构体定义是有效的：

```move
public struct Foo<T> {
    foo: u64
}
```

在建模某些概念时，这样做非常方便。以下是一个示例：

```move
module a::m {
    // 货币说明符
    public struct A {}
    public struct B {}

    // 一个通用的硬币类型，可以使用货币说明符类型进行实例化。
    //   例如 Coin<A>, Coin<B> 等等
    public struct Coin<Currency> has store {
        value: u64
    }

    // 编写关于所有货币的通用代码
    public fun mint_generic<Currency>(value: u64): Coin<Currency> {
        Coin { value }
    }

    // 编写关于某个货币具体代码
    public fun mint_a(value: u64): Coin<A> {
        mint_generic(value)
    }
    public fun mint_b(value: u64): Coin<B> {
        mint_generic(value)
    }
}
```

在此示例中，`Coin<Currency>` 是泛型的 `Currency` 类型参数，指定了硬币的货币类型，并允许代码既可以通
用地处理任何货币，也可以具体地处理特定货币。即使在 `Coin` 的任何字段中没有使用 `Currency` 类型参数，
这种通用性也适用。

### Phantom Type Parameters

In the example above, although `struct Coin` asks for the `store` ability, neither `Coin<A>` nor
`Coin<B>` will have the `store` ability. This is because of the rules for
[Conditional Abilities and Generic Types](./abilities.md#conditional-abilities-and-generic-types)
and the fact that `A` and `B` don't have the `store` ability, despite the fact that they are not
even used in the body of `struct Coin`. This might cause some unpleasant consequences. For example,
we are unable to put `Coin<A>` into a wallet in storage.

One possible solution would be to add spurious ability annotations to `A` and `B` (i.e.,
`public struct Currency1 has store {}`). But, this might lead to bugs or security vulnerabilities
because it weakens the types with unnecessary ability declarations. For example, we would never
expect a value in the storage to have a field in type `A`, but this would be possible with the
spurious `store` ability. Moreover, the spurious annotations would be infectious, requiring many
functions generic on the unused type parameter to also include the necessary constraints.

Phantom type parameters solve this problem. Unused type parameters can be marked as _phantom_ type
parameters, which do not participate in the ability derivation for structs. In this way, arguments
to phantom type parameters are not considered when deriving the abilities for generic types, thus
avoiding the need for spurious ability annotations. For this relaxed rule to be sound, Move's type
system guarantees that a parameter declared as `phantom` is either not used at all in the struct
definition, or it is only used as an argument to type parameters also declared as `phantom`.

#### Declaration

In a struct definition a type parameter can be declared as phantom by adding the `phantom` keyword
before its declaration.

```move
public struct Coin<phantom Currency> has store {
    value: u64
}
```

If a type parameter is declared as phantom we say it is a phantom type parameter. When defining a
struct, Move's type checker ensures that every phantom type parameter is either not used inside the
struct definition or it is only used as an argument to a phantom type parameter.

```move
public struct S1<phantom T1, T2> { f: u64 }
//               ^^^^^^^ valid, T1 does not appear inside the struct definition

public struct S2<phantom T1, T2> { f: S1<T1, T2> }
//               ^^^^^^^ valid, T1 appears in phantom position
```

The following code shows examples of violations of the rule:

```move
public struct S1<phantom T> { f: T }
//               ^^^^^^^ ERROR!  ^ Not a phantom position

public struct S2<T> { f: T }
public struct S3<phantom T> { f: S2<T> }
//               ^^^^^^^ ERROR!     ^ Not a phantom position
```

More formally, if a type is used as an argument to a phantom type parameter we say the type appears
in _phantom position_. With this definition in place, the rule for the correct use of phantom
parameters can be specified as follows: **A phantom type parameter can only appear in phantom
position**.

Note that specifying `phantom` is not required, but the compiler will warn if a type parameter could
be `phantom` but was not marked as such.

#### Instantiation

When instantiating a struct, the arguments to phantom parameters are excluded when deriving the
struct abilities. For example, consider the following code:

```move
public struct S<T1, phantom T2> has copy { f: T1 }
public struct NoCopy {}
public struct HasCopy has copy {}
```

Consider now the type `S<HasCopy, NoCopy>`. Since `S` is defined with `copy` and all non-phantom
arguments have `copy` then `S<HasCopy, NoCopy>` also has `copy`.

### 拥有 Ability 约束的 Phantom 类型参数

在之前的例子中,尽管`struct Coin`要求有`store`能力,但`Coin<A>`和`Coin<B>`都不会有`store`能力。这是因
为[条件能力和泛型类型](./abilities.md#conditional-abilities-and-generic-types)的规则,以及`A`和`B`没
有`store`能力,尽管它们在`struct Coin`的主体中甚至没有被使用。这可能会导致一些不愉快的后果。例如,我们
无法将`Coin<A>`放入存储中的钱包。

一个可能的解决方案是为`A`和`B`添加虚假的能力注释(例如,`public struct Currency1 has store {}`)。但是,
这可能会导致错误或安全漏洞,因为它用不必要的能力声明削弱了类型。例如,我们永远不会期望存储中的值有一
个`A`类型的字段,但有了虚假的`store`能力,这就成为可能。此外,这些虚假注释会具有传染性,要求许多使用未使
用类型参数的泛型函数也包含必要的约束。

幻象类型参数解决了这个问题。未使用的类型参数可以被标记为幻象类型参数,它们不参与结构体的能力推导。这
样,幻象类型参数的参数在推导泛型类型的能力时不会被考虑,从而避免了虚假能力注释的需要。为了使这个放宽的
规则是安全的,Move 的类型系统保证了声明为`phantom`的参数要么在结构体定义中完全不使用,要么只作为参数用
于同样声明为`phantom`的类型参数。

#### 声明

在结构体定义中,可以通过在类型参数声明前添加`phantom`关键字来将其声明为幻象类型参数。

```move
public struct Coin<phantom Currency> has store {
    value: u64
}
```

如果一个类型参数被声明为幻象,我们称之为幻象类型参数。在定义结构体时,Move 的类型检查器确保每个幻象类
型参数要么在结构体定义内部不使用,要么只作为幻象类型参数的参数使用。

```move
public struct S1<phantom T1, T2> { f: u64 }
//               ^^^^^^^ 有效,T1 在结构体定义中没有出现

public struct S2<phantom T1, T2> { f: S1<T1, T2> }
//               ^^^^^^^ 有效,T1 出现在幻象位置
```

以下代码显示了违反规则的例子:

```move
public struct S1<phantom T> { f: T }
//               ^^^^^^^ 错误!  ^ 不是幻象位置

public struct S2<T> { f: T }
public struct S3<phantom T> { f: S2<T> }
//               ^^^^^^^ 错误!     ^ 不是幻象位置
```

更正式地说,如果一个类型被用作幻象类型参数的参数,我们说该类型出现在幻象位置。有了这个定义,正确使用幻
象参数的规则可以被指定为:幻象类型参数只能出现在幻象位置。

请注意,指定`phantom`不是必需的,但如果一个类型参数可以是`phantom`但未被标记为`phantom`,编译器会发出警
告。

#### 实例化

在实例化结构体时,幻象参数的参数在推导结构体能力时被排除。例如,考虑以下代码:

```move
public struct S<T1, phantom T2> has copy { f: T1 }
public struct NoCopy {}
public struct HasCopy has copy {}
```

现在考虑类型`S<HasCopy, NoCopy>`。由于`S`定义时有`copy`能力,并且所有非幻象参数都有`copy`能力,因
此`S<HasCopy, NoCopy>`也有`copy`能力。

#### 带能力约束的幻象类型参数

能力约束和幻象类型参数是正交的特性,意味着幻象参数可以声明为带有能力约束。

```move
public struct S<phantom T: copy> {}
```

在实例化带有能力约束的幻象类型参数时,类型参数必须满足该约束,尽管该参数是幻象的。通常的限制适用,`T`只
能用具有`copy`能力的参数实例化。

## 约束

在上面的例子中,我们演示了如何使用类型参数来定义可以由调用者在稍后填充的"未知"类型。然而,这意味着类型
系统对该类型的信息很少,必须以非常保守的方式进行检查。在某种意义上,类型系统必须假设无约束泛型的最坏情
况 —— 一个没有[能力](./abilities.md)的类型。

约束提供了一种方法来指定这些未知类型具有哪些属性,以便类型系统可以允许原本不安全的操作。

### 声明约束

可以使用以下语法对类型参数施加约束:

```move
// T 是类型参数的名称
T: <ability> (+ <ability>)*
```

`<ability>`可以是四种[能力](./abilities.md)中的任何一种,一个类型参数可以同时被多个能力约束。因此,以
下都是有效的类型参数声明:

```move
T: copy
T: copy + drop
T: copy + drop + store + key
```

### 验证约束

约束在实例化点进行检查

```move
public struct Foo<T: copy> { x: T }

public struct Bar { x: Foo<u8> }
//                         ^^ 有效,u8 有 `copy` 能力

public struct Baz<T> { x: Foo<T> }
//                            ^ 错误! T 没有 'copy' 能力
```

函数也是类似的

```move
fun unsafe_consume<T>(x: T) {
    // 错误! x 没有 'drop' 能力
}

fun consume<T: drop>(x: T) {
    // 有效,x 将自动被丢弃
}

public struct NoAbilities {}

fun foo() {
    let r = NoAbilities {};
    consume<NoAbilities>(NoAbilities);
    //      ^^^^^^^^^^^ 错误! NoAbilities 没有 'drop' 能力
}
```

这里是一些类似的例子,但使用了`copy`能力:

```move
fun unsafe_double<T>(x: T) {
    (copy x, x)
    // 错误! T 没有 'copy' 能力
}

fun double<T: copy>(x: T) {
    (copy x, x) // 有效,T 有 'copy' 能力
}

public struct NoAbilities {}

fun foo(): (NoAbilities, NoAbilities) {
    let r = NoAbilities {};
    double<NoAbilities>(r)
    //     ^ 错误! NoAbilities 没有 'copy' 能力
}
```

更多信息,请参见能力部分
的[条件能力和泛型类型](./abilities.md#conditional-abilities-and-generic-types)。

## 递归限制

### 递归结构体

泛型结构体不能直接或间接地包含相同类型的字段,即使有不同的类型参数也不行。以下所有结构体定义都是无效
的:

```move
public struct Foo<T> {
    x: Foo<u64> // 错误! 'Foo' 包含 'Foo'
}

public struct Bar<T> {
    x: Bar<T> // 错误! 'Bar' 包含 'Bar'
}

// 错误! 'A' 和 'B' 形成了一个循环,这也是不允许的。
public struct A<T> {
    x: B<T, u64>
}

public struct B<T1, T2> {
    x: A<T1>
    y: A<T2>
}
```

### 高级主题: 类型级递归

Move 允许泛型函数递归调用。然而,当与泛型结构体结合使用时,这可能在某些情况下创建无限数量的类型,允许这
种情况会给编译器、虚拟机和其他语言组件增加不必要的复杂性。因此,这种递归是被禁止的。

这个限制可能在将来会放宽,但目前,以下示例应该能让你了解什么是允许的,什么是不允许的。

```move
module a::m {
    public struct A<T> {}

    // 有限数量的类型 -- 允许。
    // foo<T> -> foo<T> -> foo<T> -> ... 是有效的
    fun foo<T>() {
        foo<T>();
    }

    // 有限数量的类型 -- 允许。
    // foo<T> -> foo<A<u64>> -> foo<A<u64>> -> ... 是有效的
    fun foo<T>() {
        foo<A<u64>>();
    }
}
```

不允许:

```move
module a::m {
    public struct A<T> {}

    // 无限数量的类型 -- 不允许。
    // 错误!
    // foo<T> -> foo<A<T>> -> foo<A<A<T>>> -> ...
    fun foo<T>() {
        foo<Foo<T>>();
    }
}
```

同样,不允许:

```move
module a::n {
    public struct A<T> {}

    // 无限数量的类型 -- 不允许。
    // 错误!
    // foo<T1, T2> -> bar<T2, T1> -> foo<T2, A<T1>>
    //   -> bar<A<T1>, T2> -> foo<A<T1>, A<T2>>
    //   -> bar<A<T2>, A<T1>> -> foo<A<T2>, A<A<T1>>>
    //   -> ...
    fun foo<T1, T2>() {
        bar<T2, T1>();
    }

    fun bar<T1, T2> {
        foo<T1, A<T2>>();
    }
}
```

注意,类型级递归的检查基于对调用点的保守分析,不考虑控制流或运行时值。

```move
module a::m {
    public struct A<T> {}

    // 无限数量的类型 -- 不允许。
    // 错误!
    fun foo<T>(n: u64) {
        if (n > 0) foo<A<T>>(n - 1);
    }
}
```

上面例子中的函数在技术上对任何给定输入都会终止,因此只会创建有限数量的类型,但它仍被 Move 的类型系统认
为是无效的。
