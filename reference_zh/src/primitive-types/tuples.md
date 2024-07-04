# 元组与单位类型

Move 并未完全支持元组，这与其他将元组作为[一等公民](https://en.wikipedia.org/wiki/First-class_citizen)的语言有所不同。然而，为了支持多返回值，Move 提供了类似元组的表达式。这些表达式在运行时不会生成具体的值（字节码中不存在元组），因此它们有很大的局限性：

- 只能出现在表达式中（通常在函数的返回位置）。
- 不能绑定到局部变量。
- 不能存储在结构体中。
- 元组类型不能用于实例化泛型。

类似地，[unit `()`](https://en.wikipedia.org/wiki/Unit_type) 类型是 Move 源语言为了基于表达式的设计而创建的。单位值 `()` 在运行时不会产生任何值。可以将单位 `()` 视为一个空元组，适用于所有对元组的限制。

考虑到这些限制，在语言中使用元组可能会感到奇怪。但在其他语言中，元组最常见的用例之一是允许函数返回多个值。一些语言通过强迫用户编写包含多个返回值的结构体来解决这个问题。然而，在 Move 中，你不能在[结构体](../structs.md)中放置引用。这要求 Move 支持多返回值。在字节码层面，这些多返回值全部压入堆栈。在源代码层面，这些多返回值使用元组表示。

## 字面量

元组通过在括号内使用逗号分隔的表达式列表创建。

| 语法           | 类型                                                                           | 描述                                        |
| -------------- | ----------------------------------------------------------------------------- | ------------------------------------------- |
| `()`           | `(): ()`                                                                      | 单位类型，空元组，或 0 元素的元组            |
| `(e1, ..., en)`| `(e1, ..., en): (T1, ..., Tn)` 其中 `e_i: Ti` 满足 `0 < i <= n` 且 `n > 0`    | `n` 元组，`n` 元素的元组，包含 `n` 个元素   |

注意 `(e)` 并没有类型 `(e): (t)`，换句话说，不存在单元素元组。如果括号内只有一个元素，则括号仅用于消除歧义，没有其他特殊含义。

有时，包含两个元素的元组称为"对"，包含三个元素的元组称为"三元组"。

### 示例

```move
module 0x42::example {
    // 以下三个函数是等价的

    // 当没有提供返回类型时，假定为 `()`
    fun returns_unit_1() { }

    // 空表达式块中有一个隐式的 () 值
    fun returns_unit_2(): () { }

    // 显式版本的 `returns_unit_1` 和 `returns_unit_2`
    fun returns_unit_3(): () { () }

    fun returns_3_values(): (u64, bool, address) {
        (0, false, @0x42)
    }
    fun returns_4_values(x: &u64): (&u64, u8, u128, vector<u8>) {
        (x, 0, 1, b"foobar")
    }
}
```

## 操作

目前，对元组唯一可以执行的操作是解构。

### 解构

对于任何大小的元组，都可以在 `let` 绑定或赋值中解构。

例如：

```move
module 0x42::example {
    // 以下三个函数是等价的
    fun returns_unit() {}
    fun returns_2_values(): (bool, bool) { (true, false) }
    fun returns_4_values(x: &u64): (&u64, u8, u128, vector<u8>) { (x, 0, 1, b"foobar") }

    fun examples(cond: bool) {
        let () = ();
        let (mut x, mut y): (u8, u64) = (0, 1);
        let (mut a, mut b, mut c, mut d) = (@0x0, 0, false, b"");

        () = ();
        (x, y) = if (cond) (1, 2) else (3, 4);
        (a, b, c, d) = (@0x1, 1, true, b"1");
    }

    fun examples_with_function_calls() {
        let () = returns_unit();
        let (mut x, mut y): (bool, bool) = returns_2_values();
        let (mut a, mut b, mut c, mut d) = returns_4_values(&0);

        () = returns_unit();
        (x, y) = returns_2_values();
        (a, b, c, d) = returns_4_values(&1);
    }
}
```

更多详情请参见 [Move 变量](../variables.md)。

## 子类型

与引用一样，元组是 Move 中唯一具有[子类型](https://en.wikipedia.org/wiki/Subtyping)的类型。元组的子类型仅在引用中的协变方式存在。

例如：

```move
let x: &u64 = &0;
let y: &mut u64 = &mut 1;

// (&u64, &mut u64) 是 (&u64, &u64) 的子类型
// 因为 &mut u64 是 &u64 的子类型
let (a, b): (&u64, &u64) = (x, y);

// (&mut u64, &mut u64) 是 (&u64, &u64) 的子类型
// 因为 &mut u64 是 &u64 的子类型
let (c, d): (&u64, &u64) = (y, y);

// 错误! (&u64, &mut u64) 不是 (&mut u64, &mut u64) 的子类型
// 因为 &u64 不是 &mut u64 的子类型
let (e, f): (&mut u64, &mut u64) = (x, y);
```

## 所有权

如前所述，元组值在运行时并不真正存在。目前它们不能存储到局部变量中（但未来可能会添加此功能）。因此，元组只能移动，不能复制，因为复制它们需要首先将其放入局部变量中。