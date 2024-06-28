# 字符串

虽然 Move 没有内置的字符串类型，但它在[标准库](./standard-library.md)中提供了两种字符串的标准实现。`std::string` 模块定义了一个 `String` 类型和处理 UTF-8 编码字符串的方法，而第二个模块 `std::ascii` 则提供了 ASCII `String` 类型及其方法。

> Sui 执行环境会自动将字节向量转换为事务输入中的 `String`。因此，在许多情况下，不需要在[事务块](./../concepts/what-is-a-transaction.md)中构造字符串。

<!--

## 字节字符串字面量

TODO：

- 引用向量
- 引用字面量 - [表达式](./expression.md#literals)

-->

## 字符串即字节

无论使用哪种类型的字符串，重要的是要知道字符串只是字节。`string` 和 `ascii` 模块提供的封装只是封装而已。它们提供了安全检查和与字符串相关的方法，但归根结底，它们只是字节的向量。

```move
{{#include ../../../packages/samples/sources/move-basics/string.move:custom}}
```

## 使用 UTF-8 字符串

虽然标准库中有两种类型的字符串，但应该将 `string` 模块视为默认选项。它具有许多常见操作的本地实现，因此比完全在 Move 中实现的 `ascii` 模块更高效。

### 定义

`std::string` 模块中的 `String` 类型定义如下：

```move
// 文件：move-stdlib/sources/string.move
/// `String` 保存一个字节序列，该序列保证是 utf8 格式的。
public struct String has copy, drop, store {
    bytes: vector<u8>,
}
```

### 创建字符串

要创建一个新的 UTF-8 `String` 实例，可以使用 `string::utf8` 方法。为了方便起见，[标准库](./standard-library.md)还为 `vector<u8>` 提供了别名 `.to_string()`。

```move
{{#include ../../../packages/samples/sources/move-basics/string.move:utf8}}
```

### 常见操作

UTF8 字符串提供了许多用于操作字符串的方法。字符串的常见操作包括连接、切片和获取长度。另外，对于自定义的字符串操作，可以使用 `bytes()` 方法获取底层字节向量。

```move
let mut str = b"Hello,".to_string();
let another = b" World!".to_string();

// append(String) 将内容添加到字符串的末尾
str.append(another);

// `sub_string(start, end)` 复制字符串的一个子串
str.sub_string(0, 5); // "Hello"

// `length()` 返回字符串中的字节数
str.length(); // 12 (字节)

// 方法也可以链式调用！获取子串的长度
str.sub_string(0, 5).length(); // 5 (字节)

// 字符串是否为空
str.is_empty(); // false

// 获取底层字节向量以进行自定义操作
let bytes: &vector<u8> = str.bytes();
```

### 安全的 UTF-8 操作

默认的 `utf8` 方法在传入的字节不是有效的 UTF-8 时可能会中止。如果不确定传入的字节是否有效，应改用 `try_utf8` 方法。它返回一个 `Option<String>`，如果字节不是有效的 UTF-8，则不包含值，否则包含一个字符串。

> 提示：以 `try_*` 开头的名称表示函数返回一个包含期望结果的 Option，如果操作失败，则返回 `none`。这是从 Rust 借用的常见命名约定。

```move
{{#include ../../../packages/samples/sources/move-basics/string.move:safe_utf8}}
```

### UTF-8 的限制

`string` 模块没有提供一种访问字符串中单个字符的方法。这是因为 UTF-8 是一种可变长度编码，并且一个字符的长度可以从 1 到 4 个字节不等。类似地，`length()` 方法返回的是字符串的字节数，而不是字符数。

然而，`sub_string` 和 `insert` 等方法会检查字符边界，并在索引位于字符中间时中止。

## ASCII 字符串

本节内容即将推出！