# 测试

测试是软件开发中至关重要的一环，尤其是在区块链开发中更为重要。
在这里，我们将介绍 Move 语言的测试基础知识，并讲解如何编写和组织 Move 代码的测试。

## `#[test]` 属性

在 Move 中，测试是使用 `#[test]` 属性标记的函数。
这个属性告诉编译器该函数是一个测试函数，应该在执行测试时运行。
测试函数是常规函数，但不能接受任何参数，也不能有返回值。它们不会被编译成字节码，也不会被发布。

```move
module book::testing {
    // `#[test]` 属性放置在 `fun` 关键字之前。
    // 可以放在函数签名的上方或紧挨着 `fun` 关键字：`#[test] fun my_test() { ... }`
    // 测试的名称将会是 `book::testing::simple_test`。
    #[test]
    fun simple_test() {
        let sum = 2 + 2;
        assert!(sum == 4, 1);
    }

    // 测试的名称将会是 `book::testing::more_advanced_test`.
    #[test] fun more_advanced_test() {
        let sum = 2 + 2 + 2;
        assert!(sum == 4, 1);
    }
}
```

## 运行测试

要运行测试，可以使用 `sui move test` 命令。
该命令首先会在 _测试模式_ 下构建包，然后运行包中所有找到的测试。
在测试模式下，`sources/` 和 `tests/` 目录中的模块都会被处理，测试也会被执行。

```bash
$ sui move test
> 包含依赖项 Sui
> 包含依赖项 MoveStdlib
> 构建 book
> 运行 Move 单元测试
> ...
```

<!-- TODO: fill output -->

## 使用 `#[expected_failure]` 处理测试失败的情况

针对失败情况的测试可以使用 `#[expected_failure]` 标记。
将此属性放在 `#[test]` 函数上，告知编译器该测试预期会失败。
当你想测试某个函数在特定条件下会失败时，这个功能非常有用。

> 该属性只能放在 `#[test]` 函数上。

该属性可以接受一个终止码作为参数，即测试失败时预期的终止码。
如果测试以不同的终止码失败，测试将失败。
如果执行过程中没有终止，测试同样会失败。

```move
module book::testing_failure {

    const EInvalidArgument: u64 = 1;

    #[test]
    #[expected_failure(abort_code = 0)]
    fun test_fail() {
        abort 0 // 以终止码 0 终止
    }

    // 属性可以组合在一起使用。
    #[test, expected_failure(abort_code = EInvalidArgument)]
    fun test_fail_1() {
        abort 1 // 以终止码 1 终止
    }
}
```

`abort_code` 参数可以使用在测试模块中定义的常量，也可以从其他模块导入的常量。
在这种情况下，是唯一可以在其他模块中使用和“访问”常量的场景。

## 使用 `#[test_only]` 标记的工具函数

在某些情况下，让测试环境访问一些内部函数或特性是有帮助的。
它可以简化测试过程，并允许更全面的测试。
然而，需要注意的是，这些函数不应包含在最终的包中。
这时，`#[test_only]` 属性就派上用场了。

```move
module book::testing {
    // 使用 `secret` 函数的公共函数
    public fun multiply_by_secret(x: u64): u64 {
        x * secret()
    }

    /// 私有函数不能被公共函数使用
    fun secret(): u64 { 100 }

    #[test_only]
    /// 此函数仅用于测试中的测试目的以及其他仅限测试的函数。
    /// 注意可见性——对于 `#[test_only]`，
    /// 通常使用 `public` 可见性。
    public fun secret_for_testing(): u64 {
        secret()
    }

    #[test]
    // 在测试环境中，我们可以访问 `secret_for_testing` 函数。
    fun test_multiply_by_secret() {
        let expected = secret_for_testing() * 2;
        assert!(multiply_by_secret(2) == expected, 1);
    }
}
```

使用 `#[test_only]` 标记的函数将在测试环境中可用，
如果它们的可见性设置允许，其他模块也可以访问这些函数。

## 进一步阅读

- [单元测试](https://move-book.com/reference/unit-testing.html) 在 Move 参考手册中.