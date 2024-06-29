## 所有权和作用域

Move 中的每个变量都拥有一个作用域和一个所有者。作用域是变量有效的代码范围，所有者是该变量所属的范围。一旦所有者作用域结束，变量就会被丢弃。这是 Move 的一个基本概念，理解它的工作原理非常重要。

## 所有权

在函数作用域中定义的变量由该作用域所有。运行时会遍历函数作用域并执行每个表达式和语句。一旦函数作用域结束，定义在其中的变量就会被丢弃或释放。

```move
module book::ownership {
    public fun owner() {
        let a = 1; // a 由 `owner` 函数拥有
    } // a 在此处被丢弃

    public fun other() {
        let b = 2; // b 由 `other` 函数拥有
    } // b 在此处被丢弃

    #[test]
    fun test_owner() {
        owner();
        other();
        // a 和 b 在此处无效
    }
}
```

在上面的例子中，变量 `a` 由 `owner` 函数拥有，变量 `b` 由 `other` 函数拥有。当调用这些函数中的每一个时，都会定义变量，当函数结束后，变量就会被丢弃。

## 返回值

如果我们将 `owner` 函数改为返回变量 `a`，那么 `a` 的所有权将被转移到函数的调用者。

```move
module book::ownership {
    public fun owner(): u8 {
        let a = 1; // a 在此处定义
        a // 作用域结束，a 被返回
    }

    #[test]
    fun test_owner() {
        let a = owner();
        // a 在此处有效
    } // a 在此处被丢弃
}
```

## 按值传递

此外，如果我们将变量 `a` 传递给另一个函数，则 `a` 的所有权将被转移到该函数。执行此操作时，我们将值从一个作用域 _移动_ 到另一个作用域。这也被称为 _move 语义_。

```move
module book::ownership {
    public fun owner(): u8 {
        let a = 10;
        a
    } // a 被返回

    public fun take_ownership(v: u8) {
        // v 由 `take_ownership` 拥有
    } // v 在此处被丢弃

    #[test]
    fun test_owner() {
        let a = owner();
        take_ownership(a);
        // a 在此处无效
    }
}
```

## 具有块的作用域

每个函数都有一个主作用域，还可以通过使用块来拥有子作用域。块是一系列语句和表达式，它有自己的作用域。在块中定义的变量由该块拥有，当块结束时，变量将被丢弃。

```move
module book::ownership {
    public fun owner() {
        let a = 1; // a 由 `owner` 函数的作用域拥有
        {
            let b = 2; // b 由块拥有
            {
                let c = 3; // c 由块拥有
            }; // c 在此处被丢弃
        }; // b 在此处被丢弃
        // a = b; // 错误：b 在此处无效
        // a = c; // 错误：c 在此处无效
    } // a 在此处被丢弃
}
```

但是，如果我们使用块的返回值，则变量的所有权将被转移到块的调用者。

```move
module book::ownership {
    public fun owner(): u8 {
        let a = 1; // a 由 `owner` 函数的作用域拥有
        let b = {
            let c = 2; // c 由块拥有
            c // c 被返回
        }; // c 在此处被丢弃
        a + b // a 和 b 在此处都有效
    }
}
```

## 可复制类型

Move 中的一些类型是可复制的，这意味着它们可以被复制而不转移所有权。这对于那些小而易于复制的类型（例如整数和布尔值）非常有用。Move 编译器会在将它们传递给函数或从函数返回时，或者当它们被 _移动_ 到一个作用域然后在它们原来的作用域中访问时自动复制这些类型。

## 进一步阅读

- Move 参考中的 [局部变量和作用域](/reference/variables.html)。