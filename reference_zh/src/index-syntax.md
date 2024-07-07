# Index Syntax

Move 提供了语法属性，允许您定义操作，使这些操作看起来和感觉像原生 Move 代码，从而将这些操作降低到您提供的定义中。

我们的第一个语法方法 `index` 允许您定义一组操作，可以用作自定义索引访问器，例如通过注释应该用于这些索引操作的函数来访问矩阵元素 `m[i,j]`。此外，这些定义是每种类型专属的，并且可以隐式地供任何使用您的类型的程序员使用。

## 概述和总结

首先，考虑一个使用向量的向量表示其值的 `Matrix` 类型。您可以使用 `index` 语法注解在 `borrow` 和 `borrow_mut` 函数上编写一个小型库，如下所示：

```move
module matrix {

    public struct Matrix<T> { v: vector<vector<T>> }

    #[syntax(index)]
    public fun borrow<T>(s: &Matrix<T>, i: u64, j: u64): &T {
        vector::borrow(vector::borrow(&s.v, i), j)
    }

    #[syntax(index)]
    public fun borrow_mut<T>(s: &mut Matrix<T>, i: u64, j: u64): &mut T {
        vector::borrow_mut(vector::borrow_mut(&mut s.v, i), j)
    }

    public fun make_matrix<T>(v: vector<vector<T>>):  Matrix<T> {
        Matrix { v }
    }

}
```

现在，任何使用此 `Matrix` 类型的人都可以使用其索引语法：

```move
let mut m = matrix::make_matrix(vector[
    vector[1, 0, 0],
    vector[0, 1, 0],
    vector[0, 0, 1],
]);

let mut i = 0;
while (i < 3) {
    let mut j = 0;
    while (j < 3) {
        if (i == j) {
            assert!(m[i, j] == 1, 1);
        } else {
            assert!(m[i, j] == 0, 0);
        };
        *(&mut m[i,j]) = 2;
        j = j + 1;
    };
    i = i + 1;
}
```

## 用法

如示例所示，如果定义了数据类型和相关的索引语法方法，任何人都可以通过在该类型的值上编写索引语法来调用该方法：

```move
let mat = matrix::make_matrix(...);
let m_0_0 = mat[0, 0];
```

在编译期间，编译器会根据表达式的位置和可变性将这些转换为相应的函数调用：

```move
let mut mat = matrix::make_matrix(...);

let m_0_0 = mat[0, 0];
// 翻译为 `copy matrix::borrow(&mat, 0, 0)`

let m_0_0 = &mat[0, 0];
// 翻译为 `matrix::borrow(&mat, 0, 0)`

let m_0_0 = &mut mat[0, 0];
// 翻译为 `matrix::borrow_mut(&mut mat, 0, 0)`
```

您还可以将索引表达式与字段访问混合使用：

```move
public struct V { v: vector<u64> }

public struct Vs { vs: vector<V> }

fun borrow_first(input: &Vs): &u64 {
    &input.vs[0].v[0]
    // 翻译为 `vector::borrow(&vector::borrow(&input.vs, 0).v, 0)`
}
```

### 索引函数接受灵活参数

请注意，除了本章其余部分描述的定义和类型限制外，Move 不对您的索引语法方法接受的参数值施加限制。这允许您在定义索引语法时实现复杂的程序行为，例如一个在索引超出范围时接受默认值的数据结构：

```move
#[syntax(index)]
public fun borrow_or_set<Key: copy, Value: drop>(
    input: &mut MTable<Key, Value>,
    key: Key,
    default: Value
): &mut Value {
    if (contains(input, key)) {
        borrow(input, key)
    } else {
        insert(input, key, default);
        borrow(input, key)
    }
}
```

现在，当您索引 `MTable` 时，还必须提供一个默认值：

```move
let string_key: String = ...;
let mut table: MTable<String, u64> = m_table::make_table();
let entry: &mut u64 = &mut table[string_key, 0];
```

这种扩展能力允许您为您的类型编写精确的索引接口，从而具体执行自定义行为。

## 定义索引语法函数

这种强大的语法形式允许您定义的所有数据类型都以这种方式运行，假设您的定义遵循以下规则：

1. `#[syntax(index)]` 属性添加到在与主题类型相同的模块中定义的指定函数中。
2. 指定的函数具有 `public` 可见性。
3. 函数接受一个引用类型作为其主题类型（其第一个参数），并返回一个匹配的引用类型（如果主题是 `mut`，则返回 `mut`）。
4. 每种类型仅有一个可变和一个不可变的定义。
5. 不可变和可变版本具有类型一致性：
   - 主题类型匹配，仅在可变性上有所不同。
   - 返回类型与其主题类型的可变性匹配。
   - 如果存在类型参数，则在两个版本之间具有相同的约束。
   - 除主题类型外的所有参数都相同。

以下内容和附加示例更详细地描述了这些规则。

### 声明

要声明索引语法方法，请在相关函数定义上方添加 `#[syntax(index)]` 属性，该函数定义在与主题类型定义相同的模块中。这向编译器表明该函数是指定类型的索引访问器。

#### 不可变访问器

不可变索引语法方法是为只读访问定义的。它接受主题类型的不可变引用，并返回元素类型的不可变引用。在 `std::vector` 中定义的 `borrow` 函数是一个示例：

```move
#[syntax(index)]
public native fun borrow<Element>(v: &vector<Element>, i: u64): &Element;
```

#### 可变访问器

可变索引语法方法是不可变方法的对偶，允许进行读写操作。它接受主题类型的可变引用，并返回元素类型的可变引用。在 `std::vector` 中定义的 `borrow_mut` 函数是一个示例：

```move
#[syntax(index)]
public native fun borrow_mut<Element>(v: &mut vector<Element>, i: u64): &mut Element;
```

#### 可见性

为了确保索引函数在类型使用的任何地方都可用，所有索引语法方法必须具有公共可见性。这确保了跨 Move 模块和包的索引使用的便捷性。

#### 无重复

除了上述要求外，我们还限制每个主题基类型定义一个不可变引用和一个可变引用的索引语法方法。例如，您不能为多态类型定义专门版本：

```move
#[syntax(index)]
public fun borrow_matrix_u64(s: &Matrix<u64>, i: u64, j: u64): &u64 { ... }

#[syntax(index)]
public fun borrow_matrix<T>(s: &Matrix<T>, i: u64, j: u64): &T { ... }
// 错误！Matrix 已经有了不可变索引语法方法的定义
```

这确保了您始终可以知道正在调用哪个方法，而无需检查类型实例化。


### 类型约束

默认情况下，索引语法方法具有以下类型约束：

**其主题类型（第一个参数）必须是指向同一模块中定义的单个类型的引用**。这意味着您不能为元组、类型参数或值定义索引语法方法：

```move
#[syntax(index)]
public fun borrow_fst(x: &(u64, u64), ...): &u64 { ... }
    // 错误，因为主题类型是元组

#[syntax(index)]
public fun borrow_tyarg<T>(x: &T, ...): &T { ... }
    // 错误，因为主题类型是类型参数

#[syntax(index)]
public fun borrow_value(x: Matrix<u64>, ...): &u64 { ... }
    // 错误，因为x不是引用
```

**主题类型必须与返回类型具有相同的可变性。** 这个限制允许您在借用索引表达式作为`&vec[i]`和`&mut vec[i]`时，明确预期的行为。Move编译器使用可变性标记来确定调用哪种借用形式以生成相应可变性的引用。因此，我们不允许主题和返回可变性不同的索引语法方法：

```move
#[syntax(index)]
public fun borrow_imm(x: &mut Matrix<u64>, ...): &u64 { ... }
    // 错误！不兼容的可变性
    // 预期的是可变引用 '&mut' 返回类型
```

### 类型兼容性

当定义一个不可变和可变的索引语法方法对时，它们需要满足一些兼容性约束：

1. 它们必须接受相同数量的类型参数，并且这些类型参数必须具有相同的约束。
1. 类型参数必须按位置而不是名称使用。
1. 除了可变性之外，它们的主题类型必须完全匹配。
1. 除了可变性之外，它们的返回类型必须完全匹配。
1. 所有其他参数类型必须完全匹配。

这些约束旨在确保无论是在可变还是不可变位置，索引语法的行为都是相同的。

为了说明其中一些错误，请回顾之前的`Matrix`定义：

```move
#[syntax(index)]
public fun borrow<T>(s: &Matrix<T>, i: u64, j: u64): &T {
    vector::borrow(vector::borrow(&s.v, i), j)
}
```

以下所有定义的可变版本都是类型不兼容的：

```move
#[syntax(index)]
public fun borrow_mut<T: drop>(s: &mut Matrix<T>, i: u64, j: u64): &mut T { ... }
    // 错误！这里`T`有`drop`，但不在不可变版本中

#[syntax(index)]
public fun borrow_mut(s: &mut Matrix<u64>, i: u64, j: u64): &mut u64 { ... }
    // 错误！这里使用了不同数量的类型参数

#[syntax(index)]
public fun borrow_mut<T, U>(s: &mut Matrix<U>, i: u64, j: u64): &mut U { ... }
    // 错误！这里使用了不同数量的类型参数

#[syntax(index)]
public fun borrow_mut<U>(s: &mut Matrix<U>, i_j: (u64, u64)): &mut U { ... }
    // 错误！这里使用了不同数量的参数

#[syntax(index)]
public fun borrow_mut<U>(s: &mut Matrix<U>, i: u64, j: u32): &mut U { ... }
    // 错误！`j`是不同的类型
```

再次强调，目标是使不可变版本和可变版本之间的使用方式一致。这样一来，索引语法方法可以在可变和不可变使用时都能正常工作，而不会根据可变性改变行为或约束，最终确保具有一致的可编程接口。