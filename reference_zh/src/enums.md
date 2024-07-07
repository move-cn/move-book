# Enumerations

枚举(enum)是一种用户定义的数据结构,包含一个或多个变体(variant)。每个变体可以选择性地包含带类型的字段。这些字段的数量和类型可以在枚举的各个变体之间不同。枚举中的字段可以存储任何非引用、非元组类型,包括其他结构体或枚举。

下面是 Move 中的一个简单枚举定义示例:

```move
public enum Action {
    Stop,
    Pause { duration: u32 },
    MoveTo { x: u64, y: u64 },
    Jump(u64),
}
```

这定义了一个名为`Action`的枚举,表示游戏中可以采取的不同动作 —— 你可以`Stop`(停止),`Pause`(暂停)一段时间,`MoveTo`(移动到)特定位置,或者`Jump`(跳跃)到特定高度。

与结构体类似,枚举也可以拥有[能力](./abilities.md),这些能力控制可以对枚举执行哪些操作。需要注意的是,枚举不能拥有`key`能力,因为它们不能作为顶级对象。

## 定义枚举

枚举必须在模块中定义,一个枚举必须至少包含一个变体,每个变体可以没有字段、有位置字段或命名字段。以下是一些示例:

```move
module a::m {
    public enum Foo has drop {
        VariantWithNoFields,
        //                 ^ 注意:变体声明后可以有一个尾随逗号
    }
    public enum Bar has copy, drop {
        VariantWithPositionalFields(u64, bool),
    }
    public enum Baz has drop {
        VariantWithNamedFields { x: u64, y: bool, z: Bar },
    }
}
```

枚举在任何变体中都不能递归,所以以下枚举定义是不允许的,因为它们至少在一个变体中是递归的。

错误示例:

```move
module a::m {
    public enum Foo {
        Recursive(Foo),
        //        ^ 错误:递归枚举变体
    }
    public enum List {
        Nil,
        Cons { head: u64, tail: List },
        //                      ^ 错误:递归枚举变体
    }
    public enum BTree<T> {
        Leaf(T),
        Node { left: BTree<T>, right: BTree<T> },
        //           ^ 错误:递归枚举变体
    }

    // 相互递归的枚举也是不允许的
    public enum MutuallyRecursiveA {
        Base,
        Other(MutuallyRecursiveB),
        //    ^^^^^^^^^^^^^^^^^^ 错误:递归枚举变体
    }

    public enum MutuallyRecursiveB {
        Base,
        Other(MutuallyRecursiveA),
        //    ^^^^^^^^^^^^^^^^^^ 错误:递归枚举变体
    }
}
```

## 可见性

所有枚举都被声明为`public`。这意味着枚举的类型可以从任何其他模块引用。但是,枚举的变体、每个变体中的字段以及创建或销毁枚举变体的能力仅限于定义该枚举的模块内部。

### 能力

就像结构体一样,默认情况下枚举声明是线性和短暂的。要以非线性或非短暂的方式使用枚举值 —— 即复制、丢弃或存储在[对象](./abilities/object.md)中 —— 你需要通过用`has <ability>`注释来授予它额外的[能力](./abilities.md):

```move
module a::m {
    public enum Foo has copy, drop {
        VariantWithNoFields,
    }
}
```

能力声明可以在枚举变体之前或之后出现,但只能使用其中一种方式,不能两种都用。如果在变体之后声明,能力声明必须以分号结束:

```move
module a::m {
    public enum PreNamedAbilities has copy, drop { Variant }
    public enum PostNamedAbilities { Variant } has copy, drop;
    public enum PostNamedAbilitiesInvalid { Variant } has copy, drop
    //                                                              ^ 错误! 缺少分号

    public enum NamedInvalidAbilities has copy { Variant } has drop;
    //                                                     ^ 错误! 重复的能力声明
}
```

更多详情,请参阅[注释能力](./abilities.md#annotating-structs-and-enums)部分。

## 命名

枚举和枚举内的变体必须以大写字母`A`到`Z`开头。在第一个字母之后,枚举名可以包含下划线`_`、小写字母`a`到`z`、大写字母`A`到`Z`或数字`0`到`9`。

```move
public enum Foo { Variant }
public enum BAR { Variant }
public enum B_a_z_4_2 { V_a_riant_0 }
```

这种以`A`到`Z`开头的命名限制是为了给未来的语言特性留出空间。

## 使用枚举

### 创建枚举变体

可以通过指定枚举的一个变体,然后为该变体中的每个字段提供一个值来创建(或"打包")枚举类型的值。变体名称必须始终由枚举名称限定。

与结构体类似,对于具有命名字段的变体,字段的顺序并不重要,但需要提供字段名称。对于具有位置字段的变体,字段的顺序很重要,必须与变体声明中的顺序匹配。它还必须使用`()`而不是`{}`来创建。如果变体没有字段,变体名称就足够了,不需要使用`()`或`{}`。

```move
module a::m {
    public enum Action has drop {
        Stop,
        Pause { duration: u32 },
        MoveTo { x: u64, y: u64 },
        Jump(u64),
    }
    public enum Other has drop {
        Stop(u64),
    }

    fun example() {
        // 注意: `Action`的`Stop`变体没有字段,所以不需要括号或大括号。
        let stop = Action::Stop;
        let pause = Action::Pause { duration: 10 };
        let move_to = Action::MoveTo { x: 10, y: 20 };
        let jump = Action::Jump(10);
        // 注意: `Other`的`Stop`变体确实有位置字段,所以我们需要提供它们。
        let other_stop = Other::Stop(10);
    }
}
```

对于具有命名字段的变体,你也可以使用你可能从结构体中熟悉的简写语法来创建变体:

```move
let duration = 10;

let pause = Action::Pause { duration: duration };
// 等同于
let pause = Action::Pause { duration };
```

### 枚举变体和解构的模式匹配

由于枚举值可以采用不同的形式，因此不允许像结构字段那样直接访问变体的字段。相反，要访问变体内部的字段（无论是通过值、不可变引用还是可变引用），您必须使用模式匹配。

在Move中，可以通过值、不可变引用和可变引用进行模式匹配。通过值进行模式匹配时，该值被移动到匹配的分支中。通过引用进行模式匹配时，该值被借用到匹配的分支中（可以是不可变的或可变的）。我们在这里简要介绍使用 `match` 进行模式匹配，但如果想了解更多关于Move中使用 `match` 进行模式匹配的信息，请参阅[模式匹配](./control-flow/pattern_matching.md)部分。

`match`语句用于对Move值进行模式匹配，由多个匹配分支组成。每个匹配分支由一个模式、一个箭头 `=>` 和一个表达式组成，后跟逗号 `,`。模式可以是结构体、枚举变体、绑定（`x`、`y`）、通配符（`_`或`..`）、常量（`ConstValue`）或文字值（`true`、`42`等）。值将从上到下依次与每个模式进行匹配，并匹配第一个结构上匹配的模式。一旦值匹配成功，就会执行 `=>` 右侧的表达式。

此外，匹配分支可以有可选的 _条件_，在模式匹配成功后但在执行表达式之前进行检查。条件由 `if` 关键字指定，后跟必须评估为布尔值的表达式。

下面是一个使用枚举和模式匹配的示例，展示了如何根据不同的枚举变体执行不同的操作：

```move
module a::m {
    public enum Action has drop {
        Stop,
        Pause { duration: u32 },
        MoveTo { x: u64, y: u64 },
        Jump(u64),
    }

    public struct GameState {
        // 游戏状态包含的字段
        character_x: u64,
        character_y: u64,
        character_height: u64,
        // ...
    }

    fun perform_action(state: &mut GameState, action: Action) {
        match action {
            // 处理 `Stop` 变体
            Action::Stop => state.stop(),
            // 处理 `Pause` 变体
            // 如果持续时间为 0，则什么也不做
            Action::Pause { duration: 0 } => (),
            Action::Pause { duration } => state.pause(duration),
            // 处理 `MoveTo` 变体
            Action::MoveTo { x, y } => state.move_to(x, y),
            // 处理 `Jump` 变体
            // 如果游戏不允许跳跃，则什么也不做
            Action::Jump(_) if state.jumps_not_allowed() => (),
            Action::Jump(height) => state.jump(height),
        }
    }
}
```

接下来，我们看看如何在枚举上进行模式匹配，以在可变情况下更新其值。我们将以一个简单的枚举为例，其中有两个变体，每个变体都有一个字段。然后编写两个函数，一个仅递增第一个变体的值，另一个仅递增第二个变体的值：

```move
module a::m {
    public enum SimpleEnum {
        Variant1(u64),
        Variant2(u64),
    }

    public fun incr_enum_variant1(simple_enum: &mut SimpleEnum) {
        match simple_enum {
            SimpleEnum::Variant1(value) => *value += 1,
            _ => (),
        }
    }

    public fun incr_enum_variant2(simple_enum: &mut SimpleEnum) {
        match simple_enum {
            SimpleEnum::Variant2(value) => *value += 1,
            _ => (),
        }
    }
}
```

现在，如果有一个 `SimpleEnum` 的值，我们可以使用这些函数来递增该变体的值：

```move
let mut x = SimpleEnum::Variant1(10);
incr_enum_variant1(&mut x);
assert!(x == SimpleEnum::Variant1(11));
// 由于它递增了不同的变体，因此不会递增
incr_enum_variant2(&mut x);
assert!(x == SimpleEnum::Variant1(11));
```

当在Move值上进行模式匹配时，如果值没有 `drop` 能力，则必须在每个匹配分支中消耗或解构该值。如果在匹配分支中未消耗或解构值，则编译器会引发错误。这是为了确保在匹配语句中处理了所有可能的值。

例如，考虑以下代码：

```move
module a::m {
    public enum X { Variant { x: u64 } }

    public fun bad(x: X) {
        match x {
            _ => ()
           // ^ 错误！在此匹配分支中未消耗或解构类型为 `X` 的值
        }
    }
}
```

要正确处理这种情况，您需要在匹配的分支中解构 `X` 及其所有变体：

```move
module a::m {
    public enum X { Variant { x: u64 } }

    public fun good(x: X) {
        match x {
            // OK！编译通过，因为值已解构
            X::Variant { x: _ } => ()
        }
    }
}
```

### 覆盖枚举值

只要枚举具有 `drop` 能力，您就可以像在Move中处理其他值一样，使用新类型的枚举值覆盖枚举的值。

```move
module a::m {
    public enum X has drop {
        A(u64),
        B(u64),
    }

    public fun overwrite_enum(x: &mut X) {
        *x = X::A(10);
    }
}
```

```move
let mut x = X::B(20);
overwrite_enum(&mut x);
assert!(x == X::A(10));
```
