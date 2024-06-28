# 更好的错误处理

当执行过程中遇到中止时，交易将失败，并将中止代码返回给调用者。Move VM 返回中止交易的模块名称和中止代码。此行为对交易的调用者并不完全透明，尤其是在单个函数包含对可能中止的相同函数的多个调用时。在这种情况下，调用者将不知道哪个调用中止了交易，并且难以调试问题或向用户提供有意义的错误消息。

```move
module book::module_a {
    use book::module_b;

    public fun do_something() {
        let field_1 = module_b::get_field(1); // 可能中止并返回 0
        /* ... 许多逻辑 ... */
        let field_2 = module_b::get_field(2); // 可能中止并返回 0
        /* ... 更多逻辑 ... */
        let field_3 = module_b::get_field(3); // 可能中止并返回 0
    }
}
```

上面的例子说明了单个函数包含多个可能中止的调用的情况。如果 `do_something` 函数的调用者收到中止代码 `0`，将很难理解是哪个调用中止了交易。为了解决这个问题，可以使用一些常见的模式来改进错误处理。

## 规则 1：处理所有可能的情况

提供一个安全的“检查”函数以返回布尔值，指示操作是否可以安全执行是一种良好的实践。如果 `module_b` 提供了一个返回布尔值指示字段是否存在的函数 `has_field`，则可以将 `do_something` 函数重写如下：

```move
module book::module_a {
    use book::module_b;

    const ENoField: u64 = 0;

    public fun do_something() {
        assert!(module_b::has_field(1), ENoField);
        let field_1 = module_b::get_field(1);
        /* ... */
        assert!(module_b::has_field(2), ENoField);
        let field_2 = module_b::get_field(2);
        /* ... */
        assert!(module_b::has_field(3), ENoField);
        let field_3 = module_b::get_field(3);
    }
}
```

通过在每次调用 `module_b::get_field` 之前添加自定义检查，`module_a` 的开发者控制了错误处理。这也使得实现第二条规则成为可能。

## 规则 2：使用不同的代码中止

一旦调用者模块处理了中止代码，使用不同的中止代码针对不同的情况。这使得调用者模块可以向用户提供有意义的错误消息。可以将 `module_a` 重写如下：

```move
module book::module_a {
    use book::module_b;

    const ENoFieldA: u64 = 0;
    const ENoFieldB: u64 = 1;
    const ENoFieldC: u64 = 2;

    public fun do_something() {
        assert!(module_b::has_field(1), ENoFieldA);
        let field_1 = module_b::get_field(1);
        /* ... */
        assert!(module_b::has_field(2), ENoFieldB);
        let field_2 = module_b::get_field(2);
        /* ... */
        assert!(module_b::has_field(3), ENoFieldC);
        let field_3 = module_b::get_field(3);
    }
}
```

现在，调用者模块可以向用户提供有意义的错误消息。如果调用者收到中止代码 `0`，可以将其翻译为“字段 1 不存在”。如果调用者收到中止代码 `1`，可以将其翻译为“字段 2 不存在”，依此类推。

## 规则 3：返回布尔值而不是断言

开发人员常常倾向于添加一个公共函数来断言所有条件并中止执行。然而，更好的做法是创建一个返回布尔值的函数。这样，调用者模块可以处理错误并向用户提供有意义的错误消息。

```move
module book::some_app_assert {

    const ENotAuthorized: u64 = 0;

    public fun do_a() {
        assert_is_authorized();
        // ...
    }

    public fun do_b() {
        assert_is_authorized();
        // ...
    }

    /// 不要这样做
    public fun assert_is_authorized() {
        assert!(/* 一些条件 */ true, ENotAuthorized);
    }
}
```

此模块可以重写如下：

```move
module book::some_app {
    const ENotAuthorized: u64 = 0;

    public fun do_a() {
        assert!(is_authorized(), ENotAuthorized);
        // ...
    }

    public fun do_b() {
        assert!(is_authorized(), ENotAuthorized);
        // ...
    }

    public fun is_authorized(): bool {
        /* 一些条件 */ true
    }

    // 私有函数仍可以用于避免代码重复的情况
    // 当相同的条件与相同的中止代码在多个地方使用时
    fun assert_is_authorized() {
        assert!(is_authorized(), ENotAuthorized);
    }
}
```

利用这三个规则可以使交易调用者的错误处理更加透明，并允许其他开发人员在其模块中使用自定义中止代码。