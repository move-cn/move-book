# 已废弃: Friends

注意: 这个功能已被 [`public(package)`](./functions.md#visibility) 所取代。

`friend` 语法曾用于声明当前模块信任的其他模块。受信任的模块可以调用当前模块中定义的任何具有 `public(friend)` 可见性的函数。有关函数可见性的详细信息,请参阅 [Functions](./functions.md) 中的"可见性"部分。

## Friend 声明

模块可以通过 friend 声明语句将其他模块声明为朋友,格式如下:

- `friend <address::name>` — 使用完全限定的模块名称进行 friend 声明,如下例所示:

  ```move
  module 0x42::a {
      friend 0x42::b;
  }
  ```

- `friend <module-name-alias>` — 使用模块名称别名进行 friend 声明,其中模块别名通过 `use` 语句引入。

  ```move
  module 0x42::a {
      use 0x42::b;
      friend b;
  }
  ```

一个模块可以有多个 friend 声明,所有 friend 模块的并集形成 friend 列表。在下面的例子中,`0x42::B` 和 `0x42::C` 都被视为 `0x42::A` 的朋友。

```move
module 0x42::a {

    friend 0x42::b;
    friend 0x42::c;
}
```

与 `use` 语句不同,`friend` 只能在模块作用域中声明,不能在表达式块作用域中声明。`friend` 声明可以位于允许顶级构造(如 `use`、`function`、`struct` 等)的任何位置。但是,为了提高可读性,建议将 friend 声明放在模块定义的开头附近。

### Friend 声明规则

Friend 声明受以下规则约束:

- 模块不能将自己声明为朋友。

  ```move
  module 0x42::m { friend Self; // 错误! }
  //                      ^^^^ 不能将模块自身声明为朋友

  module 0x43::m { friend 0x43::M; // 错误! }
  //                      ^^^^^^^ 不能将模块自身声明为朋友
  ```

- Friend 模块必须为编译器所知。

  ```move
  module 0x42::m { friend 0x42::nonexistent; // 错误! }
  //                      ^^^^^^^^^^^^^^^^^ 未绑定的模块 '0x42::nonexistent'
  ```

- Friend 模块必须在同一账户地址内。

  ```move
  module 0x42::m {}

  module 0x42::n { friend 0x42::m; // 错误! }
  //                      ^^^^^^^ 不能将当前地址以外的模块声明为朋友
  ```

- Friends 关系不能创建循环模块依赖。

  Friend 关系中不允许出现循环,例如,`0x2::a` 友好 `0x2::b` 友好 `0x2::c` 友好 `0x2::a` 这样的关系是不允许的。更一般地说,声明一个 friend 模块会在当前模块和 friend 模块之间添加一个依赖关系(因为目的是让 friend 调用当前模块中的函数)。如果该 friend 模块已经被直接或间接使用,就会创建一个依赖循环。

  ```move
  module 0x2::a {
      use 0x2::c;
      friend 0x2::b;

      public fun a() {
          c::c()
      }
  }

  module 0x2::b {
      friend 0x2::c; // 错误!
  //         ^^^^^^ 这个 friend 关系创建了一个依赖循环: '0x2::b' 是 '0x2::a' 的朋友,使用了 '0x2::c',而 '0x2::c' 又是 '0x2::b' 的朋友
  }

  module 0x2::c {
      public fun c() {}
  }
  ```

- 模块的 friend 列表不能包含重复项。

  ```move
  module 0x42::a {}

  module 0x42::m {
      use 0x42::a as aliased_a;
      friend 0x42::A;
      friend aliased_a; // 错误!
  //         ^^^^^^^^^ 重复的 friend 声明 '0x42::a'。模块中的 friend 声明必须是唯一的
  }
  ```