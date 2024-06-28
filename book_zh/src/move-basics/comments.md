# 注释

<!--

章节：基本语法
目标：介绍注释。
注意：
    - 文档注释用于生成文档
    - 只有公共成员会被记录
    - 文档注释放在属性和定义之间
    - 文档注释适用于：模块、结构体、函数、常量
    - 给出文档注释如何被翻译的示例
 -->

注释是一种为代码添加注释或文档的方法。它们会被编译器忽略，不会生成 Move 字节码。你可以使用注释来解释代码的功能，向自己或其他开发者添加备注，暂时移除部分代码或生成文档。Move 中有三种类型的注释：行注释、块注释和文档注释。

## 行注释

```Move
{{#include ../../../packages/samples/sources/move-basics/comments.move:line}}
```

你可以使用双斜杠 `//` 来注释掉余下的行。编译器会忽略 `//` 之后的所有内容。

```Move
{{#include ../../../packages/samples/sources/move-basics/comments.move:line_2}}
```

## 块注释

块注释用于注释掉一段代码。它们以 `/*` 开始，以 `*/` 结束。编译器会忽略 `/*` 和 `*/` 之间的所有内容。你可以使用块注释来注释掉单行或多行代码，甚至可以注释掉一行中的一部分。

```Move
{{#include ../../../packages/samples/sources/move-basics/comments.move:block}}
```

这个例子有点极端，但它展示了如何使用块注释来注释掉一行中的一部分。

## 文档注释

文档注释是一种特殊的注释，用于为代码生成文档。它们类似于块注释，但以三个斜杠 `///` 开始，并放在它们所记录的项目定义之前。

```Move
{{#include ../../../packages/samples/sources/move-basics/comments.move:doc}}
```

<!-- TODO: docgen, 哪些成员会在文档中 -->
