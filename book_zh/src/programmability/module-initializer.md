# 模块初始化器

在许多应用程序中，一个常见的用例是仅在发布包时运行某些代码。想象一个简单的商店模块，需要在发布时创建主商店对象。在Sui中，这可以通过在模块中定义一个`init`函数来实现。当模块发布时，该函数将自动被调用。

> 所有模块的`init`函数都会在发布过程中被调用。目前，这种行为仅限于发布命令，不包括[包升级](./package-upgrades.md)。

```move
{{#include ../../../packages/samples/sources/programmability/module-initializer.move:main}}
```

在同一个包中，另一个模块可以有自己的`init`函数，封装不同的逻辑。

```move
{{#include ../../../packages/samples/sources/programmability/module-initializer.move:other}}
```

## `init`功能

如果模块中存在`init`函数并遵循以下规则，则在发布时会调用该函数：

- 函数必须命名为`init`，是私有的并且没有返回值。
- 接受一个或两个参数：[一次性见证](./one-time-witness.md)（可选）和[交易上下文](./transaction-context.md)。`TxContext`始终是最后一个参数。

```move
fun init(ctx: &mut TxContext) { /* ... */ }
fun init(otw: OTW, ctx: &mut TxContext) { /* ... */ }
```

TxContext也可以作为不可变引用传递：`&TxContext`。然而，实际上，它应该始终是`&mut TxContext`，因为`init`函数不能访问链上状态，并且要创建新对象需要上下文的可变引用。

```move
fun init(ctx: &TxContext) { /* ... */ }
fun init(otw: OTW, ctx: &TxContext) { /* ... */ }
```

## 信任与安全

虽然`init`函数可以用于创建一次性对象，但重要的是要知道同样的对象（例如第一个示例中的`StoreOwnerCap`）仍然可以在另一个函数中创建。特别是考虑到在升级过程中可以向模块添加新函数。因此，`init`函数是设置模块初始状态的好地方，但它本身不是一种安全措施。

有一些方法可以保证对象只创建一次，例如[一次性见证](./one-time-witness.md)。还有一些方法可以限制或禁用模块的升级，我们将在[包升级](./package-upgrades.md)章节中进行讨论。

## 下一步

根据定义，`init`函数保证在模块发布时只调用一次。因此，它是放置初始化模块对象、设置环境和配置的代码的好地方。

例如，如果有一个[Capability](./capability.md)，需要它来执行某些操作，那么它应该在`init`函数中创建。在下一章节中，我们将更详细地讨论`Capability`模式。