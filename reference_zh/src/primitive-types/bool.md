# 布尔类型

`bool` 是 Sui Move 语言中用于表示布尔值 `true` 和 `false` 的原始类型。

## 字面值

布尔类型的字面值可以是 `true` 或 `false`。

## 操作

### 逻辑运算

`bool` 支持三种逻辑运算：

| 语法                      | 描述                       | 等效表达式                                                      |
| ------------------------- | -------------------------- | --------------------------------------------------------------- |
| `&&`                      | 短路逻辑与                  | `p && q` 等效于 `if (p) q else false`                             |
| <code>&vert;&vert;</code> | 短路逻辑或                  | <code>p &vert;&vert; q</code> 等效于 `if (p) true else q`         |
| `!`                       | 逻辑非                      | `!p` 等效于 `if (p) false else true`                              |

### 控制流

`bool` 值在 Sui Move 的多个控制流结构中使用：

- [`if (bool) { ... }`](../control-flow/conditionals.md)
- [`while (bool) { .. }`](../control-flow/loops.md)
- [`assert!(bool, u64)`](../abort-and-assert.md)

## 所有权

与语言中其他标量值一样，布尔值是隐式可复制的，这意味着它们可以在不需要显式指令（如 [`copy`](../variables.md#move-and-copy)）的情况下进行复制。