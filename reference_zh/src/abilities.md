# 能力

能力是Move语言中的一个类型特性，用于控制某种类型的值可以执行哪些操作。这套系统提供了对值的“线性”类型行为的细粒度控制，以及值在存储中的使用方式（由Move的具体部署定义，例如区块链中的存储概念）。这是通过限制对某些字节码指令的访问来实现的，只有当一个值具有所需的能力（如果需要的话——并非每个指令都需要能力）时，才能使用这些字节码指令。

对于Sui，`key`用于表示一个[对象](./abilities/object.md)。对象是存储的基本单位，每个对象都有一个唯一的32字节ID。`store`则用于指示可以存储在对象内部的数据类型，同时也用于指示哪些类型可以在定义模块之外进行转移。

## 四种能力

四种能力分别是：

- [`copy`](#copy)
  - 允许具有此能力的类型的值被复制。
- [`drop`](#drop)
  - 允许具有此能力的类型的值被丢弃。
- [`store`](#store)
  - 允许具有此能力的类型的值存在于存储中的某个值内。
  - 对于Sui，`store`控制哪些数据可以存储在[对象](./abilities/object.md)内，也控制哪些类型可以在定义模块之外转移。
- [`key`](#key)
  - 允许类型作为存储的“键”。这意味着该值可以作为存储中的顶级值；换句话说，它不需要包含在其他值中即可存在于存储中。
  - 对于Sui，`key`用于表示一个[对象](./abilities/object.md)。

### `copy`

`copy`能力允许具有此能力的类型的值被复制。它限制了从局部变量中复制值的能力，通过[`copy`](./variables.md#move-and-copy)操作符以及通过引用复制值的能力，通过[dereference `*e`](./primitive-types/references.md#reading-and-writing-through-references)操作符。

如果一个值具有`copy`，那么该值内部包含的所有值都具有`copy`。

### `drop`

`drop`能力允许具有此能力的类型的值被丢弃。丢弃是指该值未被转移且在Move程序执行时被有效地销毁。因此，这种能力限制了在多个位置忽略值的能力，包括：

- 不使用局部变量或参数中的值
- 不使用[序列中的值通过`;`](./variables.md#expression-blocks)
- 在[赋值](./variables.md#assignments)时覆盖变量中的值
- 在[引用写操作时](./primitive-types/references.md#reading-and-writing-through-references)覆盖值。

如果一个值具有`drop`，那么该值内部包含的所有值都具有`drop`。

### `store`

`store`能力允许具有此能力的类型的值存在于存储中的某个值内，但不一定作为存储中的顶级值。这是唯一一个不直接限制操作的能力。相反，它在与`key`一起使用时限制了在存储中的存在。

如果一个值具有`store`，那么该值内部包含的所有值都具有`store`。

对于Sui，`store`有双重作用。它控制哪些值可以出现在一个[对象](./abilities/object.md)内部，以及哪些对象可以在定义模块之外[转移](./abilities/object.md#transfer-rules)。

### `key`

`key`能力允许类型作为存储操作的键，如Move的部署所定义。虽然它特定于每个Move实例，但它用于限制所有存储操作，因此要想使用存储原语，类型必须具有`key`能力。

如果一个值具有`key`，那么该值内部包含的所有值都具有`store`。这是唯一具有这种不对称性的能力。

对于Sui，`key`用于表示一个[对象](./abilities/object.md)。

## 内建类型

所有原始内建类型都具有`copy`、`drop`和`store`。

- `bool`、`u8`、`u16`、`u32`、`u64`、`u128`、`u256`和`address`都具有`copy`、`drop`和`store`。
- `vector<T>`可能具有`copy`、`drop`和`store`，这取决于`T`的能力。
  - 参见[条件能力和泛型类型](#conditional-abilities-and-generic-types)了解更多详情。
- 不可变引用`&`和可变引用`&mut`都具有`copy`和`drop`。
  - 这指的是复制和丢弃引用本身，而不是它们引用的内容。
  - 引用不能出现在全局存储中，因此它们不具有`store`。

注意，原始类型中没有一个具有`key`，这意味着它们不能直接用于存储操作。

## 标注结构体和枚举

要声明一个结构体或枚举具有某种能力，可以在数据类型名称之后、字段/变体之前或之后使用`has <ability>`进行声明。例如：

```move
{{#include ../../packages/reference/sources/abilities.move:annotating_datatypes}}
```

在这种情况下：`Ignorable*`具有`drop`能力。`Pair*`和`MyVec*`都具有`copy`、`drop`和`store`。

所有这些能力对这些受限制的操作具有强保证。只有当值具有这种能力时，才能对该值执行操作；即使该值深深嵌套在某个集合中也是如此！

因此：在声明结构体的能力时，对字段有一定的要求。所有字段必须满足这些约束。这些规则是必要的，以便结构体满足上述能力的可达性规则。如果一个结构体被声明具有能力...

- `copy`，所有字段必须具有`copy`。
- `drop`，所有字段必须具有`drop`。
- `store`，所有字段必须具有`store`。
- `key`，所有字段必须具有`store`。
  - `key`是目前唯一不要求自身的能力。

枚举可以具有上述任何能力，除了`key`，因为枚举不能作为存储中的顶级值（对象）。对于枚举变体的字段，规则与结构体字段相同。如果枚举被声明具有能力...

- `copy`，所有变体的所有字段必须具有`copy`。
- `drop`，所有变体的所有字段必须具有`drop`。
- `store`，所有变体的所有字段必须具有`store`。
- `key`，枚举不允许具有此能力。

例如：

```move
// 一个没有任何能力的结构体
public struct NoAbilities {}

public struct WantsCopy has copy {
    f: NoAbilities, // 错误 'NoAbilities' 不具有 'copy'
}

public enum WantsCopyEnum has copy {
    Variant1
    Variant2(NoAbilities), // 错误 'NoAbilities' 不具有 'copy'
}
```

同样地：

```move
// 一个没有任何能力的结构体
public struct NoAbilities {}

public struct MyData has key {
    f: NoAbilities, // 错误 'NoAbilities' 不具有 'store'
}

public struct MyDataEnum has store {
    Variant1,
    Variant2(NoAbilities), // 错误 'NoAbilities' 不具有 'store'
}
```

## 条件能力和泛型类型

当能力标注在泛型类型上时，并不是该类型的所有实例都保证具有该能力。考虑以下结构体声明：

```move
{{#include ../../packages/reference/sources/abilities.move:conditional_abilities}}
```

如果`Cup`能够容纳任何类型，而不考虑其能力，那将非常有帮助。类型系统可以_看到_类型参数，因此如果它_看到_一个类型参数会违反该能力的保证，它应该能够从`Cup`中删除该能力。

这种行为一开始可能听起来有点混乱，但如果我们考虑集合类型，它可能会更容易理解。我们可以将内建类型`vector`看作具有以下类型声明：

```move
vector<T> has copy, drop, store;
```

我们希望`vector`能与任何类型一起工作。我们不希望为不同的能力创建单独的`vector`类型。那么我们希望的规则是什么？正是我们在上述字段规则中希望的规则。因此，只有当内部元素可以复制时，才可以复制`vector`值。只有当内部元素可以忽略/丢弃时，才可以忽略`vector`值。而且，只有当内部元素可以存在于存储中时，才可以将`vector`放入存储中。

为了具有这种额外的表达能力，类型可能没有它声明的所有能力，具体取决于该类型的实例化；相反，一个类型将具有的能力取决于其声明**和**其类型参数。对于任何类型，类型参数被悲观地假设用于结构体内部，因此只有当类型参数满足上述字段的要求时，才授予能力。以上面的`Cup`为例：

- 只有当`T`具有`copy`时，`Cup`才具有`copy`能力。
- 只有当`T

`具有`drop`时，它才具有`drop`能力。
- 只有当`T`具有`store`时，它才具有`store`能力。
- 只有当`T`具有`store`时，它才具有`key`能力。

以下是每种能力的条件系统示例：

### 示例：条件`copy`

```move
public struct NoAbilities {}
public struct S has copy, drop { f: bool }
public struct Cup<T> has copy, drop, store { item: T }

fun example(c_x: Cup<u64>, c_s: Cup<S>) {
    // 有效，'Cup<u64>'具有'copy'，因为'u64'具有'copy'
    let c_x2 = copy c_x;
    // 有效，'Cup<S>'具有'copy'，因为'S'具有'copy'
    let c_s2 = copy c_s;
}

fun invalid(c_account: Cup<signer>, c_n: Cup<NoAbilities>) {
    // 无效，'Cup<signer>'不具有'copy'。
    // 即使'Cup'声明了copy，实例也不具有'copy'，
    // 因为'signer'不具有'copy'
    let c_account2 = copy c_account;
    // 无效，'Cup<NoAbilities>'不具有'copy'，
    // 因为'NoAbilities'不具有'copy'
    let c_n2 = copy c_n;
}
```

### 示例：条件`drop`

```move
public struct NoAbilities {}
public struct S has copy, drop { f: bool }
public struct Cup<T> has copy, drop, store { item: T }

fun unused() {
    Cup<bool> { item: true }; // 有效，'Cup<bool>'具有'drop'
    Cup<S> { item: S { f: false }}; // 有效，'Cup<S>'具有'drop'
}

fun left_in_local(c_account: Cup<signer>): u64 {
    let c_b = Cup<bool> { item: true };
    let c_s = Cup<S> { item: S { f: false }};
    // 有效返回：'c_account'、'c_b'和'c_s'具有值
    // 但'Cup<signer>'、'Cup<bool>'和'Cup<S>'具有'drop'
    0
}

fun invalid_unused() {
    // 无效，不能忽略'Cup<NoAbilities>'，因为它不具有'drop'。
    // 即使'Cup'声明了'drop'，实例也不具有'drop'，
    // 因为'NoAbilities'不具有'drop'
    Cup<NoAbilities> { item: NoAbilities {} };
}

fun invalid_left_in_local(): u64 {
    let n = Cup<NoAbilities> { item: NoAbilities {} };
    // 无效返回：'c_n'具有一个值
    // 而'Cup<NoAbilities>'不具有'drop'
    0
}
```

### 示例: 条件性 `store` 能力

```move
public struct Cup<T> has copy, drop, store { item: T }

// 'MyInnerData' 声明时带有 'store' 能力,所以所有字段都需要 'store' 能力
struct MyInnerData has store {
    yes: Cup<u64>, // 有效,因为 'Cup<u64>' 有 'store' 能力
    // no: Cup<signer>, 无效,因为 'Cup<signer>' 没有 'store' 能力
}

// 'MyData' 声明时带有 'key' 能力,所以所有字段都需要 'store' 能力
struct MyData has key {
    yes: Cup<u64>, // 有效,因为 'Cup<u64>' 有 'store' 能力
    inner: Cup<MyInnerData>, // 有效,因为 'Cup<MyInnerData>' 有 'store' 能力
    // no: Cup<signer>, 无效,因为 'Cup<signer>' 没有 'store' 能力
}
```

### 示例: 条件性 `key` 能力

```move
public struct NoAbilities {}
public struct MyData<T> has key { f: T }

fun valid(addr: address) acquires MyData {
    // 有效,因为 'MyData<u64>' 有 'key' 能力
    transfer(addr, MyData<u64> { f: 0 });
}

fun invalid(addr: address) {
   // 无效,因为 'MyData<NoAbilities>' 没有 'key' 能力
   transfer(addr, MyData<NoAbilities> { f: NoAbilities {} })
   // 无效,因为 'MyData<NoAbilities>' 没有 'key' 能力
   borrow<NoAbilities>(addr);
   // 无效,因为 'MyData<NoAbilities>' 没有 'key' 能力
   borrow_mut<NoAbilities>(addr);
}

// 模拟存储操作
native public fun transfer<T: key>(addr: address, value: T);
```