这是一个关于Move语言中引用的详细介绍。我会将其翻译成简单易懂的中文，同时保留关键的技术术语:

# 引用

Move有两种类型的引用:不可变引用`&`和可变引用`&mut`。不可变引用是只读的,不能修改底层值(或其任何字段)。可变引用允许通过该引用进行修改。Move的类型系统强制执行所有权规则,以防止引用错误。

## 引用操作符

Move提供了创建和扩展引用以及将可变引用转换为不可变引用的操作符。这里我们用`e: T`表示"表达式`e`的类型为`T`"。

| 语法        | 类型                                   | 描述                           |
| ----------- | -------------------------------------- | ------------------------------ |
| `&e`        | `&T`,其中`e: T`且`T`不是引用类型       | 创建`e`的不可变引用            |
| `&mut e`    | `&mut T`,其中`e: T`且`T`不是引用类型   | 创建`e`的可变引用              |
| `&e.f`      | `&T`,其中`e.f: T`                      | 创建结构体`e`的字段`f`的不可变引用 |
| `&mut e.f`  | `&mut T`,其中`e.f: T`                  | 创建结构体`e`的字段`f`的可变引用 |
| `freeze(e)` | `&T`,其中`e: &mut T`                   | 将可变引用`e`转换为不可变引用  |

`&e.f`和`&mut e.f`操作符既可以用于创建新的引用到结构体中,也可以用于扩展现有引用:

```move
let s = S { f: 10 };
let f_ref1: &u64 = &s.f; // 可以
let s_ref: &S = &s;
let f_ref2: &u64 = &s_ref.f // 也可以
```

多字段的引用表达式只要两个结构体在同一模块中就可以工作:

```move
public struct A { b: B }
public struct B { c : u64 }
fun f(a: &A): &u64 {
    &a.b.c
}
```

最后,请注意不允许引用的引用:

```move
let x = 7;
let y: &u64 = &x;
let z: &&u64 = &y; // 错误! 无法编译
```

## 通过引用读写

可变和不可变引用都可以被读取以产生被引用值的副本。

只有可变引用可以被写入。写入`*x = v`会丢弃之前存储在`x`中的值,并用`v`更新它。

这两种操作都使用类似C的`*`语法。但请注意,读取是一个表达式,而写入是必须发生在等号左侧的变更。

| 语法       | 类型                                | 描述                   |
| ---------- | ----------------------------------- | ---------------------- |
| `*e`       | `T`,其中`e`是`&T`或`&mut T`         | 读取`e`指向的值        |
| `*e1 = e2` | `()`,其中`e1: &mut T`且`e2: T`      | 用`e2`更新`e1`中的值   |

为了能读取引用,底层类型必须具有[`copy`能力](../abilities.md),因为读取引用会创建值的新副本。这条规则防止了资产被复制:

```move
fun copy_coin_via_ref_bad(c: Coin) {
    let c_ref = &c;
    let counterfeit: Coin = *c_ref; // 不允许!
    pay(c);
    pay(counterfeit);
}
```

相对地:为了能写入引用,底层类型必须具有[`drop`能力](../abilities.md),因为写入引用会丢弃(或"删除")旧值。这条规则防止了资源值被销毁:

```move=
fun destroy_coin_via_ref_bad(mut ten_coins: Coin, c: Coin) {
    let ref = &mut ten_coins;
    *ref = c; // 错误! 不允许--会销毁10个硬币!
}
```

## `freeze`推断

可变引用可以在期望不可变引用的上下文中使用:

```move
let mut x = 7;
let y: &u64 = &mut x;
```

这是因为在底层,编译器会在需要的地方插入`freeze`指令。这里有更多`freeze`推断的示例:

```move
fun takes_immut_returns_immut(x: &u64): &u64 { x }

// 在返回值上进行freeze推断
fun takes_mut_returns_immut(x: &mut u64): &u64 { x }

fun expression_examples() {
    let mut x = 0;
    let mut y = 0;
    takes_immut_returns_immut(&x); // 无推断
    takes_immut_returns_immut(&mut x); // 推断为freeze(&mut x)
    takes_mut_returns_immut(&mut x); // 无推断

    assert!(&x == &mut y, 42); // 推断为freeze(&mut y)
}

fun assignment_examples() {
    let x = 0;
    let y = 0;
    let imm_ref: &u64 = &x;

    imm_ref = &x; // 无推断
    imm_ref = &mut y; // 推断为freeze(&mut y)
}
```

### 子类型

通过这种`freeze`推断,Move类型检查器可以将`&mut T`视为`&T`的子类型。如上所示,这意味着在任何使用`&T`值的表达式中,也可以使用`&mut T`值。这个术语在错误消息中用于简洁地表示在需要`&T`的地方提供了`&mut T`。例如:

```move
module a::example {
    fun read_and_assign(store: &mut u64, new_value: &u64) {
        *store = *new_value
    }

    fun subtype_examples() {
        let mut x: &u64 = &0;
        let mut y: &mut u64 = &mut 1;

        x = &mut 1; // 有效
        y = &2; // 错误! 无效!

        read_and_assign(y, x); // 有效
        read_and_assign(x, y); // 错误! 无效!
    }
}
```

将产生以下错误消息:

```text
错误:

    ┌── example.move:11:9 ───
    │
 12 │         y = &2; // 无效!
    │         ^ 对局部变量'y'的无效赋值
    ·
 12 │         y = &2; // 无效!
    │             -- 类型: '&{integer}'
    ·
  9 │         let mut y: &mut u64 = &mut 1;
    │                    -------- 不是子类型: '&mut u64'
    │

错误:

    ┌── example.move:14:9 ───
    │
 15 │         read_and_assign(x, y); // 无效!
    │         ^^^^^^^^^^^^^^^^^^^^^ 对'a::example::read_and_assign'的无效调用。参数'store'无效
    ·
  8 │         let mut x: &u64 = &0;
    │                    ---- 类型: '&u64'
    ·
  3 │     fun read_and_assign(store: &mut u64, new_value: &u64) {
    │                                -------- 不是子类型: '&mut u64'
    │
```

目前唯一具有子类型的其他类型是[元组](./tuples.md)。

## 所有权

可变和不可变引用都可以随时被复制和扩展,_即使存在同一引用的现有副本或扩展_:

```move
fun reference_copies(s: &mut S) {
  let s_copy1 = s; // 可以
  let s_extension = &mut s.f; // 也可以
  let s_copy2 = s; // 仍然可以
  ...
}
```

这可能会让熟悉Rust所有权系统的程序员感到惊讶,Rust会拒绝上面的代码。Move的类型系统在处理[复制](../variables.md#move-and-copy)时更加宽松,但在写入前确保可变引用的唯一所有权方面同样严格。

### 引用不能被存储

引用和元组是_唯一_不能作为结构体字段值存储的类型,这也意味着它们不能存在于存储或[对象](../abilities/object.md)中。在程序执行期间创建的所有引用都会在Move程序终止时被销毁;它们完全是短暂的。这也适用于所有没有`store`能力的类型:任何非`store`类型的值必须在程序终止前被销毁。[能力](../abilities.md),但请注意引用和元组更进一步,从一开始就不允许存在于结构体中。

这是Move和Rust的另一个区别,Rust允许在结构体中存储引用。

我们可以想象一个更复杂、更具表现力的类型系统,允许在结构体中存储引用。我们可以允许在没有`store`[能力](../abilities.md)的结构体中使用引用,但核心困难在于Move有一个相当复杂的系统来跟踪静态引用安全性。类型系统的这个方面也需要扩展以支持在结构体中存储引用。简而言之,Move的引用安全系统需要扩展以支持存储引用,这是我们在语言演化过程中一直在关注的问题。

<!-- TODO 实际记录借用规则的草图 -->