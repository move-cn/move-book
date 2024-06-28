# 可见性修饰符

每个模块成员都有一个可见性。默认情况下，所有模块成员都是私有的，意味着它们只能在定义它们的模块内部访问。然而，你可以添加可见性修饰符来使模块成员对外部可见，或者对同一包内的模块可见，还有一种是入口类型，这可以在事务中调用，但不能从其他模块中调用。

## 内部可见性

在没有可见性修饰符的情况下定义在模块中的函数或结构体是私有的，它们无法从其他模块中调用。

```move
module book::internal_visibility {
    // 这个函数只能在同一模块的其他函数中调用
    fun internal() { /* ... */ }

    // 同一模块 -> 可以调用 internal()
    fun call_internal() {
        internal();
    }
}
```

<!-- Move 编译器不允许此代码编译通过： -->

<!-- TODO: add failure flag to example -->

```move
module book::try_calling_internal {
    use book::internal_visibility;

    // 不同模块 -> 无法调用 internal()
    fun try_calling_internal() {
        internal_visibility::internal();
    }
}
```

## 公共可见性

通过在 `fun` 或 `struct` 关键字前添加 `public` 关键字，可以使结构体或函数变为公共可见性。

```move
module book::public_visibility {
    // 这个函数可以从其他模块中调用
    public fun public() { /* ... */ }
}
```

公共函数可以被导入并从其他模块中调用。以下代码将会编译通过：

```move
module book::try_calling_public {
    use book::public_visibility;

    // 不同模块 -> 可以调用 public()
    fun try_calling_public() {
        public_visibility::public();
    }
}
```

## 包可见性

Move 2024 引入了 _包可见性_ 修饰符。具有 _包可见性_ 的函数可以从同一包内的任何模块中调用，但不能从其他包中调用。

```move
module book::package_visibility {
    public(package) fun package_only() { /* ... */ }
}
```

包函数可以从同一包内的任何模块中调用：

```move
module book::try_calling_package {
    use book::package_visibility;

    // 同一包 `book` -> 可以调用 package_only()
    fun try_calling_package() {
        package_visibility::package_only();
    }
}
```