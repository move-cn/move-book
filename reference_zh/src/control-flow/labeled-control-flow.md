# 带标签的控制流

Move 支持在编写循环和代码块时使用带标签的控制流，允许您在循环中使用 `break` 和 `continue`，并从代码
块中 `return`（这在存在宏时特别有用）。

## 循环

循环允许您在函数中定义并转移控制到特定标签。例如，我们可以嵌套两个循环，并使用这些标签与 `break` 和
`continue` 精确指定控制流。

您可以用 `'label:` 形式为任何 `loop` 或 `while` 形式添加前缀，以便直接在那里进行跳出或继续。

为了演示这种行为，我们来考虑一个函数，它接受嵌套的数字向量（即`vector<vector<u64>>`）并根据某个阈值
进行求和，其行为如下：

- 如果所有数字的总和低于阈值，返回该总和。
- 如果将一个数字加到当前总和会超过阈值，则返回当前总和。

我们可以通过将向量的向量作为嵌套循环进行迭代，并标记外部循环来编写这个函数。如果内部循环中的任何加法
会使我们超过阈值，我们可以使用带有外部标签的`break`来一次性跳出两个循环：

```move
fun sum_until_threshold(input: &vector<vector<u64>>, threshold: u64): u64 {
    let mut sum = 0;
    let mut i = 0;
    let input_size = input.length();

    'outer: loop {
        // 跳出到outer，因为它是最近的外围循环
        if (i >= input_size) break sum;

        let vec = &input[i];
        let size = vec.length();
        let mut j = 0;

        while (j < size) {
            let v_entry = vec[j];
            if (sum + v_entry < threshold) {
                sum = sum + v_entry;
            } else {
                // 我们看到的下一个元素会突破阈值，
                // 所以我们返回当前的总和
                break 'outer sum
            };
            j = j + 1;
        };
        i = i + 1;
    }
}
```

这些标签也可以用于嵌套的循环形式，在更大的代码块中提供精确的控制。例如，如果我们正在处理一个大表，其
中每个条目都需要迭代，可能会看到我们继续内部或外部循环，我们可以使用标签来表达该代码：

```move
let x = 'outer: loop {
    ...
    'inner: while (cond) {
        ...
        if (cond0) { break 'outer value };
        ...
        if (cond1) { continue 'inner }
        else if (cond2) { continue 'outer }
        ...
    }
        ...
};
```

## 标记块 (Labeled Blocks)

标记块允许您编写包含函数内部非局部控制流的 Move 程序，包括在宏 lambda 内部和返回值时：

```move
fun named_return(n: u64): vector<u8> {
    let x = 'a: {
        if (n % 2 == 0) {
            return 'a b"even"
        };
        b"odd"
    };
    x
}
```

在这个简单的例子中，程序检查输入 `n` 是否为偶数。如果是，程序将以值 `b"even"` 离开标记为 `'a:` 的块
。如果不是，代码继续执行，以值 `b"odd"` 结束标记为 `'a:` 的块。最后，我们将 `x` 设置为该值，然后返回
它。

这个控制流特性也适用于宏体。例如，假设我们想编写一个函数来查找向量中的第一个偶数，并且我们有一些宏
`for_ref` 可以在循环中迭代向量元素：

```move
macro fun for_ref<$T>($vs: &vector<$T>, $f: |&$T|) {
    let vs = $vs;
    let mut i = 0;
    let end = vs.length();
    while (i < end) {
        $f(vs.borrow(i));
        i = i + 1;
    }
}
```

使用 `for_ref` 和一个标签，我们可以编写一个 lambda 表达式来传递 `for_ref`，它将跳出循环，返回找到的
第一个偶数：

```move
fun find_first_even(vs: vector<u64>): Option<u64> {
    'result: {
        for_ref!(&vs, |n| if (*n % 2 == 0) { return 'result option::some(*n)});
        option::none()
    }
}
```

此函数将迭代 `vs`，直到找到一个偶数，并返回该偶数（如果不存在偶数，则返回 `option::none()`）。这使得
命名标签成为与控制流宏（如 `for!`）交互的强大工具，使您能够在这些上下文中自定义迭代行为。

## 限制

为了明确程序行为，您只能在循环标签中使用 `break` 和 `continue`，而 `return` 仅适用于块标签。为此，以
下程序会产生错误：

```move
fun bad_loop() {
    'name: loop {
        return 'name 5
            // ^^^^^ Invalid usage of 'return' with a loop block label
    }
}

fun bad_block() {
    'name: {
        continue 'name;
              // ^^^^^ Invalid usage of 'break' with a loop block label
        break 'name;
           // ^^^^^ Invalid usage of 'break' with a loop block label
    }
}
```
