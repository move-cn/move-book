# 条件`if`表达式

`if`表达式指定只有在某个条件为真时才会评估一些代码。
例如：

```move
if (x > 5) x = x - 5
```

条件必须是一个`bool`类型的表达式。

`if`表达式可以可选地包含一个`else`子句，用于指定当条件为假时要评估的另一个表达式。

```move
if (y <= 10) y = y + 1 else y = 10
```

"true"分支或"false"分支将被评估，但不会同时评估两者。任一分支可以是单个表达式或表达式块。

条件表达式可以生成值，以使`if`表达式具有结果。

```move
let z = if (x < 100) x else 100;
```

true分支和false分支的表达式必须具有兼容的类型。例如：

```move=
// x 和 y 必须是 u64 整数
let maximum: u64 = if (x > y) x else y;

// 错误！分支类型不同
let z = if (maximum < 10) 10u8 else 100u64;

// 错误！分支类型不同，因为默认的false分支是()而不是u64
if (maximum >= 10) maximum;
```

如果未指定`else`子句，则false分支默认为单元值。以下两种写法是等价的：

```move
if (条件) true分支 // 默认隐含：else ()
if (条件) true分支 else ()
```

通常，`if`表达式与表达式块一起使用。

```move
let maximum = if (x > y) x else y;
if (maximum < 10) {
    x = x + 10;
    y = y + 10;
} else if (x >= 10 && y >= 10) {
    x = x - 10;
    y = y - 10;
}
```

## 条件语句的语法

> _if-expression_ → **if (** _expression_ **)** _expression_ _else-clause_<sub>_opt_</sub> >
> _else-clause_ → **else** _expression_