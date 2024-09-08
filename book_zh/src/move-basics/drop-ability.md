# 能力：Drop

<!-- TODO: reiterate, given that we introduce abilities one by one -->

<!-- TODO:

- introduce abilities first
- mention them all
- then do one by one

consistency: we / I / you ?
who is we? I am alone, there's no one else here


-->

<!-- TODO: 

// Shall we only talk about `drop` ?
// So that we don't explain scopes and `copy` / `move` semantics just yet?

Chapter: Basic Syntax
Goal: Introduce Copy and Drop abilities of Move. Follows the `struct` section
Notes:
    - compare them to primitive types introduces before;
    - what is an ability without drop
    - drop is not necessary for unpacking
    - make a joke about a bacteria pattern in the code
    - mention that a struct with only `drop` ability is called a Witness
    - mention that a struct without abilities is called a Hot Potato
    - mention that there are two more abilities which are covered in a later chapter

Links:
    - language reference (abilities)
    - authorization patterns (or witness)
    - hot potato pattern
    - key and store abilities (later chapter)

 -->

`drop` 能力是最简单的能力，允许对结构体的实例进行“忽略”或“丢弃”。在许多编程语言中，这被认为是默认行为。然而，在Move中，不允许忽略没有`drop`能力的结构体。这是Move语言的一个安全特性，它确保所有资产都得到正确处理。试图忽略没有`drop`能力的结构体将导致编译错误。

```move
{{#include ../../../packages/samples/sources/move-basics/drop-ability.move:main}}
```

`drop` 能力通常在自定义的集合类型上使用，以消除在不再需要集合时的特殊处理需求。例如，`vector` 类型具有 `drop` 能力，这使得在不再需要时可以忽略该向量。然而，Move类型系统最大的特点是能够没有 `drop`。这确保了资产得到正确处理，而不被忽略。

一个仅具有 `drop` 能力的结构体称为 _Witness_。我们在[见证和抽象实现](./../programmability/witness-and-abstract-implementation.md)部分解释了 _Witness_ 的概念。

## 带有 `drop` 能力的类型

Move中的所有原生类型都具有 `drop` 能力。包括：

- [bool](./../move-basics/primitive-types.md#booleans)
- [无符号整数](./../move-basics/primitive-types.md#integer-types)
- [vector](./../move-basics/vector.md)
- [address](./../move-basics/address.md)

标准库中定义的所有类型也都具有 `drop` 能力。包括：

- [Option](./../move-basics/option.md)
- [String](./../move-basics/string.md)
- [TypeName](./../move-basics/type-reflection.md#typename)

## 进一步阅读

- Move参考中的[Type Abilities](/reference/type-abilities.html)。