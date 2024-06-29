# 常量

<!--

Chapter: Basic Syntax
Goal: Introduce constants.
Notes:
    - constants are immutable
    - constants are private
    - start with a capital letter always
    - stored in the bytecode (but w/o a name)
    - mention standard for naming constants

Links:
    - next section (abort and assert)
    - coding conventions (constants)
    - constants (language reference)

 -->

常量是在模块级别定义的不可变值。它们通常用作给模块中使用的静态值命名的一种方式。例如，如果有一个产品的默认价格，可以为其定义一个常量。常量存储在模块的字节码中，每次使用时，值都会被复制。

```move
{{#include ../../../packages/samples/sources/move-basics/constants.move:shop_price}}
```

## 命名约定

常量必须以大写字母开头 - 这在编译器级别是强制执行的。对于用作值的常量，有一个约定，使用大写字母和下划线来分隔单词。这是一种让常量在代码中突出显示的方式。一个例外是[错误常量](./assert-and-abort.md#assert-and-abort)，它们使用ECamelCase编写。

```move
{{#include ../../../packages/samples/sources/move-basics/constants.move:naming}}
```

## 常量是不可变的

常量无法更改或赋予新值。它们是包字节码的一部分，并且是固有的不可变的。

```move
module book::immutable_constants {
    const ITEM_PRICE: u64 = 100;

    // 会产生错误
    fun change_price() {
        ITEM_PRICE = 200;
    }
}
```

## 使用配置模式

应用程序的常见用例是定义一组在整个代码库中使用的常量。但是由于常量是模块的私有内容，无法从其他模块中访问。解决这个问题的方法之一是定义一个"config"模块，导出常量。

```move
{{#include ../../../packages/samples/sources/move-basics/constants.move:config}}
```

这样，其他模块可以导入和读取常量，并简化更新过程。如果需要更改常量，只需要在包升级期间更新配置模块即可。

## 链接

- 在Move参考中的[Constants](/reference/constants.html)。
- 常量的[编码约定](./../guides/coding-conventions.md#constant)。