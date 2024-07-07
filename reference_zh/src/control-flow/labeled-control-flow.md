# 带标签的控制流

Move支持带标签的控制流，允许您在函数中定义特定标签并将控制权转移到这些标签。例如，我们可以嵌套两个循环，并使用带有这些标签的`break`和`continue`来精确指定控制流。您可以在任何`loop`或`while`形式前加上`'label:`形式，以允许直接在那里中断或继续。

为了演示这种行为，我们来考虑一个函数，它接受嵌套的数字向量（即`vector<vector<u64>>`）并根据某个阈值进行求和，其行为如下：

- 如果所有数字的总和低于阈值，返回该总和。
- 如果将一个数字加到当前总和会超过阈值，则返回当前总和。

我们可以通过将向量的向量作为嵌套循环进行迭代，并标记外部循环来编写这个函数。如果内部循环中的任何加法会使我们超过阈值，我们可以使用带有外部标签的`break`来一次性跳出两个循环：

```move
fun sum_until_threshold(input: &vector<vector<u64>>, threshold: u64): u64 {
    let mut sum = 0;
    let mut i = 0;
    let input_size = vector::length(vec);

    'outer: loop {
        // 跳出到outer，因为它是最近的外围循环
        if (i >= input_size) break sum;

        let vec = vector::borrow(input, i);
        let size = vector::length(vec);
        let mut j = 0;

        while (j < size) {
            let v_entry = *vector::borrow(vec, j);
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

这些标签也可以用于嵌套的循环形式，在更大的代码块中提供精确的控制。例如，如果我们正在处理一个大表，其中每个条目都需要迭代，可能会看到我们继续内部或外部循环，我们可以使用标签来表达该代码：

```move
'outer: loop {
    ...
    'inner: while (cond) {
        ...
        if (cond0) { break 'outer value };
        ...
        if (cond1) { continue 'inner }
        else if (cond2) { continue 'outer };
        ...
    }
    ...
}
```