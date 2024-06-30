# 纪元和时间

Sui 框架提供了两种访问当前时间的方式：`Epoch` 和 `Time`。前者表示系统中的操作期，大约每24小时更改一次。后者表示自 Unix 纪元以来的当前时间（以毫秒为单位）。在程序中可以自由访问这两个时间。

## 纪元

纪元用于将系统分隔成操作期。在一个纪元内，验证人集合是固定的；然而，在纪元边界处，验证人集合可以改变。纪元在共识算法中起着关键作用，并用于确定当前的验证人集合。它们也在质押机制中使用。

可以从[事务上下文](./transaction-context.md)中读取纪元：

```move
{{#include ../../../packages/samples/sources/programmability/epoch-and-time.move:epoch}}
```

还可以获取纪元开始时的 Unix 时间戳：

```move
{{#include ../../../packages/samples/sources/programmability/epoch-and-time.move:epoch_start}}
```

通常情况下，纪元用于质押和系统操作，然而，在自定义场景中可以使用它们来模拟24小时的周期。如果应用程序依赖于质押逻辑或需要知道当前的验证人集合，纪元非常重要。

## 时间

为了更精确地测量时间，Sui 提供了 `Clock` 对象。它是一个系统对象，在检查点期间由系统更新，它以毫秒为单位存储自 Unix 纪元以来的当前时间。`Clock` 对象在 `sui::clock` 模块中定义，并具有保留地址 `0x6`。

Clock 是一个共享对象，但是试图以可变方式访问它的事务将失败。这种限制允许对 `Clock` 对象进行并行访问，这对于保持性能是重要的。

```move
// 文件：sui-framework/clock.move
/// 单例共享对象，向 Move 调用公开时间。此对象位于地址 0x6 处，只能通过不可变引用访问（以只读方式）。
///
/// 如果尝试以可变引用或值接受 `Clock` 的入口函数将无法通过验证，而诚实的验证人将不会签署或执行使用 `Clock` 作为输入参数的事务，除非以不可变引用方式传递它。
struct Clock has key {
    id: UID,
    /// 时钟的时间戳，由系统事务在每次共识提交计划时或在测试期间通过 `sui::clock::increment_for_testing` 设置。
    timestamp_ms: u64,
}
```

`sui::clock` 模块中只有一个公共函数可用，即 `timestamp_ms`。它以毫秒为单位返回当前时间自 Unix 纪元以来的时间。

```move
{{#include ../../../packages/samples/sources/programmability/epoch-and-time.move:clock}}
```

<!-- TODO:

## Testing

TODO: how to use Clock in tests. -->