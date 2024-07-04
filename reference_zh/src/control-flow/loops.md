# Move 中的循环结构

许多程序需要对值进行迭代，Move 提供了 `while` 和 `loop` 形式来让你编写这样的代码。此外，你还可以使用 `break`（退出循环）和 `continue`（跳过本次迭代的剩余部分并返回到控制流结构的顶部）在执行过程中修改这些循环的控制流。

## `while` 循环

`while` 构造重复执行循环体（一个 `unit` 类型的表达式），直到条件（一个 `bool` 类型的表达式）计算结果为 `false`。

下面是一个简单的 `while` 循环示例，它计算从 `1` 到 `n` 的数字之和：

```move
fun sum(n: u64): u64 {
    let mut sum = 0;
    let mut i = 1;
    while (i <= n) {
        sum = sum + i;
        i = i + 1
    };

    sum
}
```

无限 `while` 循环也是允许的：

```move
fun foo() {
    while (true) { }
}
```

### 在 `while` 循环中使用 `break`

在 Move 中，`while` 循环可以使用 `break` 提前退出。例如，假设我们正在查找向量中的某个值的位置，并希望在找到它时退出：

```move
fun find_position(values: &vector<u64>, target_value: u64): Option<u64> {
    let size = vector::length(values);
    let mut i = 0;
    let mut found = false;

    while (i < size) {
        if (vector::borrow(values, i) == &target_value) {
            found = true;
            break
        };
        i = i + 1
    };

    if (found) {
        Option::Some(i)
    } else {
        Option::None
    }
}
```

在这里，如果借用的向量值等于我们的目标值，我们将 `found` 标志设置为 `true`，然后调用 `break`，这将导致程序退出循环。

最后，请注意 `while` 循环的 `break` 不能带值：`while` 循环始终返回 `unit` 类型 `()`，因此 `break` 也是。

### 在 `while` 循环中使用 `continue`

与 `break` 类似，Move 的 `while` 循环可以调用 `continue` 跳过部分循环体。这允许我们在条件不满足时跳过部分计算，例如以下示例：

```move
fun sum_even(values: &vector<u64>): u64 {
    let size = vector::length(values);
    let mut i = 0;
    let mut even_sum = 0;

    while (i < size) {
        let number = *vector::borrow(values, i);
        i = i + 1;
        if (number % 2 == 1) continue;
        even_sum = even_sum + number;
    };
    even_sum
}
```

此代码将遍历提供的向量。对于每个条目，如果该条目是偶数，它将其加到 `even_sum` 中。如果不是，则调用 `continue`，跳过求和操作并返回到 `while` 循环条件检查。

## `loop` 表达式

`loop` 表达式重复执行循环体（一个类型为 `()` 的表达式），直到遇到 `break`：

```move
fun sum(n: u64): u64 {
    let mut sum = 0;
    let mut i = 1;

    loop {
       i = i + 1;
       if (i >= n) break;
       sum = sum + i;
    };

    sum
}
```

没有 `break` 的话，循环将永远继续。在下面的示例中，由于 `loop` 没有 `break`，程序将永远运行：

```move
fun foo() {
    let mut i = 0;
    loop { i = i + 1 }
}
```

下面是一个使用 `loop` 编写 `sum` 函数的示例：

```move
fun sum(n: u64): u64 {
    let sum = 0;
    let i = 0;
    loop {
        i = i + 1;
        if (i > n) break;
        sum = sum + i
    };

    sum
}
```

### 在 `loop` 中使用带值的 `break`

与始终返回 `()` 的 `while` 循环不同，`loop` 可以使用 `break` 返回一个值。这样做时，整个 `loop` 表达式计算结果为该类型的值。例如，我们可以使用 `loop` 和 `break` 重写上面的 `find_position`，在找到目标值时立即返回索引：

```move
fun find_position(values: &vector<u64>, target_value: u64): Option<u64> {
    let size = vector::length(values);
    let mut i = 0;

    loop {
        if (vector::borrow(values, i) == &target_value) {
            break Option::Some(i)
        } else if (i >= size) {
            break Option::None
        };
        i = i + 1;
    }
}
```

此循环将以一个 Option 结果结束，并作为函数体的最后一个表达式，生成该值作为最终的函数结果。

### 在 `loop` 表达式中使用 `continue`

如你所料，`continue` 也可以在 `loop` 中使用。以下是之前使用 `loop`、`break` 和 `continue` 重写的 `sum_even` 函数。

```move
fun sum_even(values: &vector<u64>): u64 {
    let size = vector::length(values);
    let mut i = 0;
    let mut even_sum = 0;

    loop {
        if (i >= size) break;
        let number = *vector::borrow(values, i);
        i = i + 1;
        if (number % 2 == 1) continue;
        even_sum = even_sum + number;
    };
    even_sum
}
```

## `while` 和 `loop` 的类型

在 Move 中，循环是类型化表达式。`while` 表达式始终具有 `()` 类型。

```move
let () = while (i < 10) { i = i + 1 };
```

如果一个 `loop` 包含一个 `break`，该表达式的类型为 `break` 的类型。没有值的 `break` 具有 `unit` 类型 `()`。

```move
(loop { if (i < 10) i = i + 1 else break }: ());
let () = loop { if (i < 10) i = i + 1 else break };

let x: u64 = loop { if (i < 10) i = i + 1 else break 5 };
let x: u64 = loop { if (i < 10) { i = i + 1; continue} else break 5 };
```

此外，如果一个 `loop` 包含多个 `break`，它们必须全部返回相同的类型：

```move
// 无效 -- 第一个 break 返回 ()，第二个返回 5
let x: u64 = loop { if (i < 10) break else break 5 };
```

如果 `loop` 没有 `break`，`loop` 可以具有任何类型，就像 `return`、`abort`、`break` 和 `continue` 一样。

```move
(loop (): u64);
(loop (): address);
(loop (): &vector<vector<u8>>);
```

如果你需要更精确的控制流，例如跳出嵌套循环，下一章将介绍 Move 中的标签控制流的使用。