# Equality（相等性）

Move语言支持两种相等性操作 `==` 和 `!=`。

### 操作

| 语法 | 操作      | 描述                                                         |
| ---- | --------- | ------------------------------------------------------------ |
| `==` | 等于      | 如果两个操作数具有相同的值，则返回 `true`，否则返回 `false`。  |
| `!=` | 不等于    | 如果两个操作数具有不同的值，则返回 `true`，否则返回 `false`。 |

### 类型

`==` 和 `!=` 操作只有在两个操作数具有相同类型时才能使用。

```move
0 == 0; // `true`
1u128 == 2u128; // `false`
b"hello" != x"00"; // `true`
```

相等性和不相等性操作也适用于**所有**用户定义的类型！

```move
module 0x42::example {
    public struct S has copy, drop { f: u64, s: vector<u8> }

    fun always_true(): bool {
        let s = S { f: 0, s: b"" };
        s == s
    }

    fun always_false(): bool {
        let s = S { f: 0, s: b"" };
        s != s
    }
}
```

如果操作数具有不同的类型，则会出现类型检查错误：

```move
1u8 == 1u128; // 错误！
//     ^^^^^ 需要类型为 'u8' 的参数
b"" != 0; // 错误！
//     ^ 需要类型为 'vector<u8>' 的参数
```

### 引用类型比较

在比较[引用](./primitive-types/references.md)时，引用的类型（不可变或可变）并不重要。这意味着可以将一个不可变的 `&` 引用与相同底层类型的可变 `&mut` 引用进行比较。

```move
let i = &0;
let m = &mut 1;

i == m; // `false`
m == i; // `false`
m == m; // `true`
i == i; // `true`
```

以上代码等同于在需要时对每个可变引用进行显式冻结：

```move
let i = &0;
let m = &mut 1;

i == freeze(m); // `false`
freeze(m) == i; // `false`
m == m; // `true`
i == i; // `true`
```

但是，底层类型必须是相同的类型：

```move
let i = &0;
let s = &b"";

i == s; // 错误！
//   ^ 需要类型为 '&u64' 的参数
```

### 自动借用

从 Move 2024 版本开始，如果一个操作数是引用而另一个不是，`==` 和 `!=` 操作符会自动将其借用。这意味着以下代码可以正常工作而不会出现任何错误：

```move
let r = &0;

// 在所有情况下，`0` 都会自动作为 `&0` 进行借用
r == 0; // `true`
0 == r; // `true`
r != 0; // `false`
0 != r; // `false`
```

这种自动借用始终是不可变借用。

### 限制

`==` 和 `!=` 操作符在比较时会消耗值。因此，类型系统要求类型必须具有 [`drop`](./abilities.md) 能力。需要注意的是，如果没有 [`drop` 能力](./abilities.md)，所有权必须在函数结束时转移，并且此类值只能在其声明模块内显式销毁。如果直接在相等性 `==` 或不相等性 `!=` 中使用它们，将会销毁该值，从而破坏 [`drop` 能力](./abilities.md) 的安全性保证！

```move=
module 0x42::example {
    public struct Coin has store { value: u64 }
    fun invalid(c1: Coin, c2: Coin) {
        c1 == c2 // 错误！
//      ^^    ^^ 这些资产将被销毁！
    }
}
```

但是，程序员可以始终先借用该值而不是直接比较该值，并且引用类型具有 [`drop`](./abilities.md)。例如：

```move=
module 0x42::example {
    public struct Coin has store { value: u64 }
    fun swap_if_equal(c1: Coin, c2: Coin): (Coin, Coin) {
        let are_equal = &c1 == c2; // 合法，注意 `c2` 会自动被借用
        if (are_equal) (c2, c1) else (c1, c2)
    }
}
```

### 避免额外的拷贝

虽然程序员可以比较任何具有 [`drop`](./abilities.md) 的类型的值，但通常应该通过引用进行比较，以避免昂贵的拷贝操作。

```move=
let v1: vector<u8> = function_that_returns_vector();
let v2: vector<u8> = function_that_returns_vector();
assert!(copy v1 == copy v2, 42);
//      ^^^^       ^^^^
use_two_vectors(v1, v2);

let s1: Foo = function_that_returns_large_struct();
let s2: Foo = function_that_returns_large_struct();
assert!(copy s1 == copy s2, 42);
//      ^^^^       ^^^^
use_two_foos(s1, s2);
```

此代码是完全可接受的（假设 `Foo` 具有 [`drop`](./abilities.md)），但不是高效的。可以移除高亮的拷贝操作，并用借用代替：

```move=
let v1: vector<u8> = function_that_returns_vector();
let v2: vector<u8> = function_that_returns_vector();
assert!(&v1 == &v2, 42);
//      ^      ^
use_two_vectors(v1, v2);

let s1: Foo = function_that_returns_large_struct();
let s2: Foo = function_that_returns_large_struct();
assert!(&s1 == &s2, 42);
//      ^      ^
use_two_foos(s1, s2);
```

`==` 本身的效率不变，但是拷贝操作被移除，因此程序更加高效。