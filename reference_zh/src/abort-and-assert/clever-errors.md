# Clever Errors（智能错误）

Clever Errors（智能错误）是一种功能，用于在断言（assert）失败或触发 `abort` 时，提供更具信息性的错误消息。它们是源级特性（source feature），在编译后会转换成一个 `u64` 的中断码（abort code），此 `u64` 值中包含了生成可读错误信息所需的行号、常量名称以及常量值的索引。基于该转换的特性，后续需要对这个 `u64` 中断码进行处理，才能得到可读的错误消息。Sui 的 GraphQL 服务器和 Sui CLI 都会自动执行这样的后处理。如果您想手动解析 Clever 错误中断码，可参考[展开（inflate）Clever 中断码](#inflating-clever-abort-codes)的流程进行操作。

> Clever Errors 会在错误信息中包含源文件的行号等数据。因此，若源文件发生任何变动（例如自动格式化、添加新的模块成员或者新增空行），Clever Error 的实际数值都可能发生变化。

## Clever 中断码

Clever 中断码允许您将非 `u64` 类型的常量作为中断码，只需要在常量上加上 `#[error]` 属性即可。它们既可在断言（assert）语句中使用，也可直接作为 `abort` 中断码使用。

```move
module 0x42::a_module;

#[error]
const EIsThree: vector<u8> = b"The value is three";

// 若 `x` 为 3，则以 `EIsThree` 作为中断码
public fun double_except_three(x: u64): u64 {
    assert!(x != 3, EIsThree);
    x * x
}

// 始终以 `EIsThree` 作为中断码
public fun clever_abort() {
    abort EIsThree
}
```

在上述示例中，`EIsThree` 常量的类型为 `vector<u8>`，并非 `u64`。然而，由于有 `#[error]` 属性修饰，这个常量就能够用作中断码，并在运行时转换成一个 `u64` 的中断码值，其中包含：

1. 一个标记位（tag-bit），指示该中断码属于 Clever 中断码。
2. 触发中断（abort）所在行号（例如行号 7）。
3. 该常量名字在模块标识符（identifier）表中的索引（如 `EIsThree`）。
4. 该常量值在模块常量表（constant table）中的索引（如 `b"The value is three"`）。

如果调用 `double_except_three(3)`，它会以一个 `u64` 中断码的十六进制值中断，例如：

```
0x8000_0007_0001_0000
  ^       ^    ^    ^
  |       |    |    |
  |       |    |    |
  |       |    |    +-- 常量值索引 = 0 (b"The value is three")
  |       |    +-- 常量名称索引 = 1 (EIsThree)
  |       +-- 行号 = 7（触发断言的位置）
  +-- 标记位 = 0b1000_0000_0000_0000
```

再将其还原成可读的错误消息，示例如下：

```
Error from '0x42::a_module::double_except_three' (line 7), abort 'EIsThree': "The value is three"
```

具体的消息格式可能因解析工具而异，但在 `u64` 中断码结合该错误所在的模块后，就可以获取到生成类似上述人类可读错误消息所需的全部信息。

> Clever 中断码并不要求常量类型必须是 `vector<u8>`，任何在 Move 中可用的常量类型都可以搭配 `#[error]` 用作 Clever 中断码。

## 无中断码的断言

在断言（`assert!`）或 `abort` 语句中如果没有显式的中断码，将默认自动生成基于源码行号的中断码，并采用 Clever Error 的编码方式。此时，其常量名称与常量值索引将使用 `0xffff` 这两个哨兵（sentinel）值。例如：

```move
module 0x42::a_module;

#[test]
fun assert_false(x: bool) {
    assert!(false);
}

#[test]
fun abort_no_code() {
    abort
}
```

这两种情况都会产生一个 `u64` 中断码，其中包括：

1. 一个标记位（tag-bit），表明该中断码是 Clever 中断码。
2. 触发断言或 `abort` 的源码行号（例如第 6 行）。
3. 模块标识符表索引的哨兵值 `0xffff`。
4. 模块常量表索引的哨兵值 `0xffff`。

若调用 `assert_false(3)`，它会触发一个 `u64` 中断码，可能如下所示：

```
0x8000_0004_ffff_ffff
  ^       ^    ^    ^
  |       |    |    |
  |       |    |    |
  |       |    |    +-- 常量值索引 = 0xffff（哨兵值）
  |       |    +-- 常量名称索引 = 0xffff（哨兵值）
  |       +-- 行号 = 4（触发断言的位置）
  +-- 标记位 = 0b1000_0000_0000_0000
```

## Clever Errors 与宏（Macros）

Clever 中断码中的行号信息来源于中断发生的源码位置。对于一般函数来说，这通常是触发断言或 `abort` 代码行在函数体内的行号；但对于宏（macro），则使用宏调用处的行号。这在编写宏时非常有用，因为这样用户在使用宏时，仍然能获得准确且有用的错误消息。

```move
module 0x42::macro_exporter;

public macro fun assert_false() {
    assert!(false);
}

public macro fun abort_always() {
    abort
}

public fun assert_false_fun() {
    assert!(false); // 一直中断，并使用此处的行号
}

public fun abort_always_fun() {
    abort // 一直中断，并使用此处的行号
}
```

随后在另一个模块中使用这些宏：

```move
module 0x42::user_module;

use 0x42::macro_exporter::{
    assert_false,
    abort_always,
    assert_false_fun,
    abort_always_fun
};

fun invoke_assert_false() {
    assert_false!(); // 中断时使用当前这行的行号
}

fun invoke_abort_always() {
    abort_always!(); // 中断时使用当前这行的行号
}

fun invoke_assert_false_fun() {
    assert_false_fun(); // 中断时，会使用 assert_false_fun 中断言那行的行号
}

fun invoke_abort_always_fun() {
    abort_always_fun(); // 中断时，会使用 abort_always_fun 中 `abort` 行的行号
}
```

## 展开（Inflating）Clever 中断码

具体而言，Clever 中断码的布局如下：

```
|<tagbit>|<reserved>|<source line number>|<module identifier index>|<module constant index>|
+--------+----------+--------------------+-------------------------+-----------------------+
| 1-bit  | 15-bits  |      16-bits      |         16-bits         |         16-bits       |
```

值得注意的是，Move 中断（MoveAbort）还会附带一些额外信息——对我们来说，最重要的是触发错误的模块信息。因为常量名称和常量值的索引要在该模块的标识符表和常量表中才能解析（如果它们不是哨兵值的话）。

> 若要解析一个 Clever 中断码，您需要知道该错误发生在哪个模块内（即需知道对应的 `package_id` 和 `module_name`），尤其当常量名称或常量值索引并非 `0xffff` 的情况下。

以下示例的伪代码展示了如何解析一个 Clever 中断码：

```rust
// MoveAbort 提供的一些信息
let clever_abort_code: u64 = ...;
let (package_id, module_name): (PackageStorageId, ModuleName) = ...;

let is_clever_abort = (clever_abort_code & 0x8000_0000_0000_0000) != 0;

if is_clever_abort {
    // 获取行号、标识符索引和常量索引
    // 若索引为 0xffff，表示该字段为哨兵值
    let line_number = ((clever_abort_code & 0x0000_ffff_0000_0000) >> 32) as u16;
    let identifier_index = ((clever_abort_code & 0x0000_0000_ffff_0000) >> 16) as u16;
    let constant_index = ((clever_abort_code & 0x0000_0000_0000_ffff)) as u16;

    // 打印行号错误消息
    print!("Error from '{}::{}' (line {})", package_id, module_name, line_number);

    // 若标识符与常量索引都为哨兵值，则无需再打印或加载模块信息
    if identifier_index == 0xffff && constant_index == 0xffff {
        return;
    }

    // 如果常量名称和常量值的索引都非 0xffff，则需加载模块
    let module: CompiledModule = fetch_module(package_id, module_name);

    // 打印常量名称（如果存在）
    if identifier_index != 0xffff {
        let constant_name = module.get_identifier_at_table_index(identifier_index);
        print!(", '{}'", constant_name);
    }

    // 打印常量值（如果存在）
    if constant_index != 0xffff {
        let constant_value = module.get_constant_at_table_index(constant_index).deserialize_on_constant_type().to_string();
        print!(": {}", constant_value);
    }

    return;
}
```
