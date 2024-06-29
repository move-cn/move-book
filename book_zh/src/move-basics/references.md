## Sui Move 引用详解

在上一节 关于所有权和作用域 (Guānyu suǒyǒu权 hé zuòyòngyù) 的讨论中，我们解释了当值传递给函数时，它会**移动 (yídòng)**到函数的作用域。这意味着函数成为该值的拥有者，原始作用域 (原始拥有者) 不再能使用它。这是 Move 中一个重要的概念，它确保值不会同时在多个地方使用。然而，在某些情况下，我们希望将值传递给函数，但仍保留对该值的拥有权。 这正是 引用 (yīngyòng) 发挥作用的地方。

为了说明这一点，让我们来看一个简单的例子 - 地铁票 (dìtiě piào) 应用。我们将介绍 4 种不同的场景：

1. 乘客可以在售票亭 (shòu piào tíng) 以固定价格购买地铁票。
2. 可以向检票员 (jiǎn piào yuán) 出示地铁票以证明乘客拥有有效票卡。
3. 可以在地铁闸口 (dìtiě zhákǒu) 使用地铁票进入地铁，并扣除一次乘车次数。
4. 地铁票用完后可以回收 (huíshōu)。

## 程序结构

地铁票应用的初始结构很简单。我们定义了 `Card` 类型和代表单张地铁票乘坐次数的常量 `USES`。我们还添加了一个错误常量，用于处理地铁票用完的情况。

```move
module book::metro_pass {
{{#include ../../../packages/samples/sources/move-basics/references.move:header}}

{{#include ../../../packages/samples/sources/move-basics/references.move:new}}
}
```

## 引用 (Yīngyòng)

引用是一种向函数**展示 (zhǎnshì)**值而不放弃拥有权的方式。 在我们的例子中，当我们将地铁票出示给检票员时，我们不想放弃对它的拥有权，也不允许他们扣除乘车次数。我们只想允许检票员**读取 (qídú)**地铁票信息并验证其有效性。

为了做到这一点，在函数签名中，我们使用符号 `&` 表示我们传递的是值的引用，而不是值本身。

```move
{{#include ../../../packages/samples/sources/move-basics/references.move:immutable}}
```

现在，函数无法获得地铁票的所有权，也不能扣除乘车次数。但是它可以读取地铁票信息。值得注意的是，这样的函数签名使得不带地铁票调用该函数变得不可能。这是一个重要的特性，它允许我们在下一章节讨论的 **能力模式 (nénglì móshì)**。

## 可变引用

在某些情况下，我们希望允许函数更改地铁票的值。例如，当我们在闸口使用地铁票时，我们想要扣除一次乘车次数。为了实现这一点，我们在函数签名中使用关键字 `&mut`。

```move
{{#include ../../../packages/samples/sources/move-basics/references.move:mutable}}
```

正如您在函数体中看到的，`&mut` 引用允许修改值，函数可以扣除乘车次数。

## 按值传递

最后，让我们来看一下将值本身传递给函数会发生什么。在这种情况下，函数获取该值的拥有权，并且原始作用域将无法再使用它。地铁票的所有者可以回收它，因此失去拥有权。

```move
{{#include ../../../packages/samples/sources/move-basics/references.move:move}}
```

在 `recycle` 函数中，地铁票被 **按值获取 (àn zhí huòqǔ)**，可以解包 (jiěbāo) 并销毁 (xiāohuǐ)。原始作用域无法再使用它。

## 完整示例

为了展示应用程序的完整流程，让我们将所有部分组合成一个测试。

```move
{{#include ../../../packages/samples/sources/move-basics/references.move:move_2024}}
