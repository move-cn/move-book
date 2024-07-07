这是关于Move语言中向量(vector)类型的详细介绍。我会将其翻译成简单易懂的中文，同时保留关键的技术术语:

# 向量

`vector<T>`是Move提供的唯一原始集合类型。`vector<T>`是`T`类型元素的同类集合,可以通过在"末端"推入/弹出值来增长或缩小。

`vector<T>`可以用任何类型`T`实例化。例如,`vector<u64>`,`vector<address>`,`vector<0x42::my_module::MyData>`,和`vector<vector<u8>>`都是有效的向量类型。

## 字面量

### 通用`vector`字面量

任何类型的向量都可以用`vector`字面量创建。

| 语法                  | 类型                                                                           | 描述                           |
| --------------------- | ------------------------------------------------------------------------------ | ------------------------------ |
| `vector[]`            | `vector[]: vector<T>`,其中`T`是任何单一的非引用类型                            | 空向量                         |
| `vector[e1, ..., en]` | `vector[e1, ..., en]: vector<T>`,其中`e_i: T`且`0 < i <= n`且`n > 0`           | 有`n`个元素的向量(长度为`n`)   |

在这些情况下,`vector`的类型是从元素类型或向量的使用中推断出来的。如果无法推断类型,或者为了更清晰,可以显式指定类型:

```move
vector<T>[]: vector<T>
vector<T>[e1, ..., en]: vector<T>
```

#### 向量字面量示例

```move
(vector[]: vector<bool>);
(vector[0u8, 1u8, 2u8]: vector<u8>);
(vector<u128>[]: vector<u128>);
(vector<address>[@0x42, @0x100]: vector<address>);
```

### `vector<u8>`字面量

Move中向量的一个常见用例是表示"字节数组",用`vector<u8>`表示。这些值通常用于加密目的,如公钥或哈希结果。这些值非常常见,以至于提供了特定的语法使值更易读,而不是必须使用`vector[]`来指定每个单独的`u8`值。

目前支持两种类型的`vector<u8>`字面量,_字节字符串_和_十六进制字符串_。

#### 字节字符串

字节字符串是带有`b`前缀的带引号字符串字面量,例如`b"Hello!\n"`。

这些是ASCII编码的字符串,允许使用转义序列。目前,支持的转义序列有:

| 转义序列 | 描述                           |
| -------- | ------------------------------ |
| `\n`     | 换行(或换行符)                 |
| `\r`     | 回车                           |
| `\t`     | 制表符                         |
| `\\`     | 反斜杠                         |
| `\0`     | 空字符                         |
| `\"`     | 引号                           |
| `\xHH`   | 十六进制转义,插入十六进制字节序列`HH` |

#### 十六进制字符串

十六进制字符串是带有`x`前缀的带引号字符串字面量,例如`x"48656C6C6F210A"`。

每个字节对,范围从`00`到`FF`,被解释为十六进制编码的`u8`值。所以每个字节对对应结果`vector<u8>`中的一个条目。

#### 字符串字面量示例

```move
fun byte_and_hex_strings() {
    assert!(b"" == x"", 0);
    assert!(b"Hello!\n" == x"48656C6C6F210A", 1);
    assert!(b"\x48\x65\x6C\x6C\x6F\x21\x0A" == x"48656C6C6F210A", 2);
    assert!(
        b"\"Hello\tworld!\"\n \r \\Null=\0" ==
            x"2248656C6C6F09776F726C6421220A200D205C4E756C6C3D00",
        3
    );
}
```

## 操作

`vector`通过Move标准库中的`std::vector`模块支持以下操作:

| 函数                                                       | 描述                                                         | 是否中止?                |
| ---------------------------------------------------------- | ------------------------------------------------------------ | ------------------------ |
| `vector::empty<T>(): vector<T>`                            | 创建一个可以存储`T`类型值的空向量                            | 从不                     |
| `vector::singleton<T>(t: T): vector<T>`                    | 创建一个包含`t`的大小为1的向量                               | 从不                     |
| `vector::push_back<T>(v: &mut vector<T>, t: T)`            | 将`t`添加到`v`的末尾                                         | 从不                     |
| `vector::pop_back<T>(v: &mut vector<T>): T`                | 移除并返回`v`中的最后一个元素                                | 如果`v`为空              |
| `vector::borrow<T>(v: &vector<T>, i: u64): &T`             | 返回索引`i`处的`T`的不可变引用                               | 如果`i`不在范围内        |
| `vector::borrow_mut<T>(v: &mut vector<T>, i: u64): &mut T` | 返回索引`i`处的`T`的可变引用                                 | 如果`i`不在范围内        |
| `vector::destroy_empty<T>(v: vector<T>)`                   | 删除`v`                                                      | 如果`v`不为空            |
| `vector::append<T>(v1: &mut vector<T>, v2: vector<T>)`     | 将`v2`中的元素添加到`v1`的末尾                               | 从不                     |
| `vector::contains<T>(v: &vector<T>, e: &T): bool`          | 如果`e`在向量`v`中返回true。否则,返回false                   | 从不                     |
| `vector::swap<T>(v: &mut vector<T>, i: u64, j: u64)`       | 交换向量`v`中第`i`和第`j`个索引处的元素                      | 如果`i`或`j`超出范围     |
| `vector::reverse<T>(v: &mut vector<T>)`                    | 原地反转向量`v`中元素的顺序                                  | 从不                     |
| `vector::index_of<T>(v: &vector<T>, e: &T): (bool, u64)`   | 如果`e`在索引`i`处的向量`v`中,返回`(true, i)`。否则,返回`(false, 0)` | 从不                     |
| `vector::remove<T>(v: &mut vector<T>, i: u64): T`          | 移除向量`v`的第`i`个元素,移动所有后续元素。这是O(n)操作,并保持向量中元素的顺序 | 如果`i`超出范围          |
| `vector::swap_remove<T>(v: &mut vector<T>, i: u64): T`     | 将向量`v`的第`i`个元素与最后一个元素交换,然后弹出该元素。这是O(1)操作,但不保持向量中元素的顺序 | 如果`i`超出范围          |

随着时间的推移,可能会添加更多操作。

## 示例

```move
use std::vector;

let mut v = vector::empty<u64>();
vector::push_back(&mut v, 5);
vector::push_back(&mut v, 6);

assert!(*vector::borrow(&v, 0) == 5, 42);
assert!(*vector::borrow(&v, 1) == 6, 42);
assert!(vector::pop_back(&mut v) == 6, 42);
assert!(vector::pop_back(&mut v) == 5, 42);
```

## 销毁和复制`vector`

`vector<T>`的某些行为取决于元素类型`T`的能力。例如,包含没有`drop`能力的元素的向量不能像上面示例中的`v`那样被隐式丢弃--它们必须使用`vector::destroy_empty`显式销毁。

注意,除非`vec`包含零个元素,否则`vector::destroy_empty`将在运行时中止:

```move
fun destroy_any_vector<T>(vec: vector<T>) {
    vector::destroy_empty(vec) // 删除这行将导致编译器错误
}
```

但对于丢弃包含具有`drop`能力的元素的向量不会发生错误:

```move
fun destroy_droppable_vector<T: drop>(vec: vector<T>) {
    // 有效!
    // 不需要显式做任何事来销毁向量
}
```

同样,除非元素类型具有`copy`能力,否则向量不能被复制。换句话说,当且仅当`T`具有`copy`能力时,`vector<T>`才具有`copy`能力。请注意,如果需要,它将被隐式复制:

```move
let x = vector[10];
let y = x; // 隐式复制
let z = x;
(y, z)
```

请记住,复制大向量可能很昂贵。如果这是一个问题,注释`intended`用法可以防止意外复制。例如,

```move
let x = vector[10];
let y = move x;
let z = x; // 错误! x已被移动
(y, z)
```

有关更多详细信息,请参阅[类型能力](../abilities.md)和[泛型](../generics.md)部分。

## 所有权

如[上文](#destroying-and-copying-vectors)所述,只有当元素可以被复制时,`vector`值才能被复制。在这种情况下,可以通过[`copy`](../variables.md#move-and-copy)或[解引用`*`](./references.md#reading-and-writing-through-references)进行复制。