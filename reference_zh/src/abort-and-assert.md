# Abort and Assert

[`return`](./functions.md) 和 `abort` 是两种控制流结构，用于结束执行，分别适用于当前函数和整个交易。

更多关于[`return`的信息可以在链接部分找到](./functions.md#return-expression)。

## `abort`

`abort` 是一个表达式，接受一个参数：类型为 `u64` 的 **中断码**。例如：

```move
abort 42
```

`abort` 表达式会停止当前函数的执行并回滚当前交易对状态所做的所有更改（注意，这一保证必须由 Move 的具体部署适配器来确保）。没有机制可以“捕获”或以其他方式处理 `abort`。

幸运的是，在 Move 中，交易是全有或全无的，这意味着只有当交易成功时，任何对存储的更改才会全部生效。对于 Sui，这意味着不会修改任何对象。

由于这种事务性提交更改的承诺，在 `abort` 之后不需要担心回退更改。虽然这种方法在灵活性上有所欠缺，但它非常简单且可预测。

与 [`return`](./functions.md) 类似，`abort` 适用于在某些条件无法满足时退出控制流。

在这个例子中，函数会从向量中弹出两个项目，但如果向量没有两个项目，则会提前中断

```move
use std::vector;

fun pop_twice<T>(v: &mut vector<T>): (T, T) {
    if (vector::length(v) < 2) abort 42;
    (vector::pop_back(v), vector::pop_back(v))
}
```

在控制流结构中，`abort` 更有用。例如，这个函数检查向量中的所有数字是否小于指定的 `bound`，否则中断

```move
use std::vector;
fun check_vec(v: &vector<u64>, bound: u64) {
    let i = 0;
    let n = vector::length(v);
    while (i < n) {
        let cur = *vector::borrow(v, i);
        if (cur > bound) abort 42;
        i = i + 1;
    }
}
```

### `assert`

`assert` 是由 Move 编译器提供的内置宏操作。它接受两个参数，一个类型为 `bool` 的条件和一个类型为 `u64` 的代码

```move
assert!(condition: bool, code: u64)
```

由于该操作是一个宏，必须使用 `!` 来调用。这是为了表明 `assert` 的参数是按表达式调用的。换句话说，`assert` 不是一个普通函数，在字节码级别不存在。它在编译器内被替换为

```move
if (condition) () else abort code
```

`assert` 比单独使用 `abort` 更常用。上面的 `abort` 示例可以使用 `assert` 重写

```move
use std::vector;
fun pop_twice<T>(v: &mut vector<T>): (T, T) {
    assert!(vector::length(v) >= 2, 42); // 现在使用 'assert'
    (vector::pop_back(v), vector::pop_back(v))
}
```

和

```move
use std::vector;
fun check_vec(v: &vector<u64>, bound: u64) {
    let i = 0;
    let n = vector::length(v);
    while (i < n) {
        let cur = *vector::borrow(v, i);
        assert!(cur <= bound, 42); // 现在使用 'assert'
        i = i + 1;
    }
}
```

注意，由于该操作被替换为这个 `if-else`，因此 `code` 的参数并不总是被评估。例如：

```move
assert!(true, 1 / 0)
```

不会导致算术错误，它相当于

```move
if (true) () else (1 / 0)
```

因此算术表达式从未被评估！

### Move 虚拟机中的中断码

使用 `abort` 时，了解 `u64` 代码将如何被虚拟机使用很重要。

通常，在成功执行后，Move 虚拟机及其具体部署的适配器会确定对存储所做的更改。

如果遇到 `abort`，虚拟机将改为指示错误。错误信息中将包含两条信息：

- 产生中断的模块（包/地址值和模块名称）
- 中断码。

例如

```move
module 0x2::example {
    public fun aborts() {
        abort 42
    }
}

module 0x3::invoker {
    public fun always_aborts() {
        0x2::example::aborts()
    }
}
```

如果一个交易，比如上面的函数 `always_aborts`，调用 `0x2::example::aborts`，虚拟机会产生一个错误，指示模块 `0x2::example` 和代码 `42`。

这对于在模块内将多个中断分组在一起非常有用。

在这个例子中，模块有两个单独的错误码，分别在多个函数中使用

```move
module 0x42::example {

    use std::vector;

    const EEmptyVector: u64 = 0;
    const EIndexOutOfBounds: u64 = 1;

    // 将 i 移动到 j，将 j 移动到 k，将 k 移动到 i
    public fun rotate_three<T>(v: &mut vector<T>, i: u64, j: u64, k: u64) {
        let n = vector::length(v);
        assert!(n > 0, EEmptyVector);
        assert!(i < n, EIndexOutOfBounds);
        assert!(j < n, EIndexOutOfBounds);
        assert!(k < n, EIndexOutOfBounds);

        vector::swap(v, i, k);
        vector::swap(v, j, k);
    }

    public fun remove_twice<T>(v: &mut vector<T>, i: u64, j: u64): (T, T) {
        let n = vector::length(v);
        assert!(n > 0, EEmptyVector);
        assert!(i < n, EIndexOutOfBounds);
        assert!(j < n, EIndexOutOfBounds);
        assert!(i > j, EIndexOutOfBounds);

        (vector::remove<T>(v, i), vector::remove<T>(v, j))
    }
}
```

## `abort` 的类型

`abort i` 表达式可以具有任何类型！这是因为这两种结构都会打破正常的控制流，所以它们永远不需要评估为该类型的值。

以下代码虽然没什么用处，但它们会通过类型检查

```move
let y: address = abort 0;
```

这种行为在某些情况下非常有用，例如当你有一个分支指令在某些分支上生成一个值，而在其他分支上则不生成。例如：

```move
let b =
    if (x == 0) false
    else if (x == 1) true
    else abort 42;
//       ^^^^^^^^ `abort 42` 的类型是 `bool`
```