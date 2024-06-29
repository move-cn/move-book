# 开源库

开源库是为 Move 生态系统做出贡献的绝佳方式。本指南将帮助你了解如何开源一个库，如何编写测试，以及如何为你的库编写文档。

## README

TODO: readme

## 命名地址

TODO: named address

## 生成文档

TODO: docgen

## 添加示例

在发布一个旨在使用的包（如 NFT 协议或库）时，展示如何使用该包是非常重要的。这就是示例的作用。Move 中没有专门用于示例的功能，但有一些用于标记示例的惯例。首先，只有源代码会被包含在包的字节码中，因此放在其他目录中的代码不会被包含，但会被测试！

这就是为什么将示例放在单独的 `examples/` 目录中是一个好主意。

```bash
sources/
    protocol.move
    library.move
tests/
    protocol_test.move
examples/
    my_example.move
Move.toml
```

## 标签和发布（Git）

TODO: tags and releases

## 与闭源兼容的技巧

TODO: compatibility via empty functions with signatures

通过这个指南，你可以更好地为 Move 生态系统贡献你的开源库，并确保它们易于使用和维护。