# 可升级性实践

为了讨论可升级性的最佳实践，我们首先需要了解包中哪些部分可以升级。可升级性的基本前提是，升级不应破坏与先前版本的公共兼容性。模块中可以在依赖包中使用的部分不应更改其静态签名。这适用于模块——模块不能从包中删除，公共结构体——它们可以在函数签名中使用，以及公共函数——它们可以从其他包中调用。

```move
// 模块不能从包中删除
module book::upgradable {
    // 依赖项可以更改（如果它们未在公共签名中使用）
    use std::string::String;
    use sui::event; // 可以删除

    // 公共结构体不能删除且不能更改
    public struct Book has key {
        id: UID,
        title: String,
    }

    // 公共结构体不能删除且不能更改
    public struct BookCreated has copy, drop {
        /* ... */
    }

    // 公共函数不能删除且其签名不能更改，但实现可以更改
    public fun create_book(ctx: &mut TxContext): Book {
        create_book_internal(ctx)

        // 可以删除和更改
        event::emit(BookCreated {
            /* ... */
        })
    }

    // 包可见性函数可以删除和更改
    public(package) fun create_book_package(ctx: &mut TxContext): Book {
        create_book_internal(ctx)
    }

    // 入口函数可以删除和更改，只要它们不是公共的
    entry fun create_book_entry(ctx: &mut TxContext): Book {
        create_book_internal(ctx)
    }

    // 私有函数可以删除和更改
    fun create_book_internal(ctx: &mut TxContext): Book {
        abort 0
    }
}
```

## 对象版本控制

要放弃包的先前版本，可以对对象进行版本控制。只要对象包含一个版本字段，并且使用对象的代码期望并断言特定版本，代码就可以强制迁移到新版本。通常，在升级后，可以使用管理员函数来更新共享状态的版本，以便可以使用新版本的代码，而旧版本由于版本不匹配而中止。

```move
module book::versioned_state {

    const EVersionMismatch: u64 = 0;

    const VERSION: u8 = 1;

    /// 共享状态（也可以是所有的）
    public struct SharedState has key {
        id: UID,
        version: u8,
        /* ... */
    }

    public fun mutate(state: &mut SharedState) {
        assert!(state.version == VERSION, EVersionMismatch);
        // ...
    }
}
```

## 使用动态字段进行配置版本控制

在 Sui 中有一种常见模式，允许更改对象的存储配置，同时保留相同的对象签名。这是通过保持基础对象简单且有版本，并将实际配置对象作为动态字段添加来实现的。使用这种 _锚_ 模式，可以通过包升级更改配置，同时保持相同的基础对象签名。

```move
module book::versioned_config {
    use sui::vec_map::VecMap;
    use std::string::String;

    /// 基础对象
    public struct Config has key {
        id: UID,
        version: u16
    }

    /// 实际配置
    public struct ConfigV1 has store {
        data: Bag,
        metadata: VecMap<String, String>
    }

    // ...
}
```

## 模块化架构

此部分即将推出！

通过这些实践，你可以确保你的 Move 包在升级时保持兼容性，同时利用灵活的版本控制机制来管理对象和配置的变化。