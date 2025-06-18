# 代码质量检查清单

Move 语言及其生态系统的快速发展使得许多旧的实践已经过时。本指南可作为一份检查清单，供开发人员审查其代码并确保其符合当前 Move 开发的最佳实践。请仔细阅读，并将尽可能多的建议应用到您的代码中。

## 代码组织

本指南中提到的一些问题，可以通过使用
[Move Formatter](https://www.npmjs.com/package/@mysten/prettier-plugin-move) CLI 工具,
或者 [CI 检查](https://github.com/marketplace/actions/move-formatter), 或
[VSCode 或(Cursor)插件](https://marketplace.visualstudio.com/items?itemName=mysten.prettier-move)来修复.

## Package Manifest

### 使用正确的版本

本指南中的所有功能都需要 Move 2024 版本，并且必须在软件包清单中指定。

```ini
[package]
name = "my_package"
edition = "2024.beta" # or (just) "2024"
```

### 隐式框架依赖

从 Sui 1.45 开始，您不再需要在 `Move.toml` 中指定框架依赖关系了 :

```ini
# old, pre 1.45
[dependencies]
Sui = { ... }

# 从现在起, Sui, Bridge, MoveStdlib and SuiSystem 已经是隐式导入的了!
[dependencies]
```

### 命名地址要添加前缀

如果您的包有一个通用名称（如`token`），尤其是如果您的项目包含多个包，请确保在命名地址中添加一个前缀：

```ini
# bad! not indicative of anything, and can conflict
[addresses]
math = "0x0"

# good! clearly states project, unlikely to conflict
[addresses]
my_protocol_math = "0x0"
```

## Imports, Module 和 Constants

### 使用 Module 标签

```move
// bad: 传统方式增加了缩进
module my_package::my_module {
    public struct A {}
}

// good!
module my_package::my_module;

public struct A {}
```

### `Self` 不要单独出现在 `use` 语句中

```move
// correct, member + self import
use my_package::other::{Self, OtherMember};

// bad! `{Self}` 单独的Self是 冗余的
use my_package::my_module::{Self};

// good!
use my_package::my_module;
```

### 使用 `use` 时将 `Self` 和其他导入成员放在一块，不要分开 

```move
// bad!
use my_package::my_module;
use my_package::my_module::OtherMember;

// good!
use my_package::my_module::{Self, OtherMember};
```

### 错误常量要使用 `EPascalCase` 命名

```move
// bad! 全部大写时用于常规变量
const NOT_AUTHORIZED: u64 = 0;

// good! clear indication it's an error constant
const ENotAuthorized: u64 = 0;
```

### 除了错误常量外其他常量要全部大写

```move
// bad! PascalCase is associated with error consts
const MyConstant: vector<u8> = b"my const";

// good! clear indication that it's a constant value
const MY_CONSTANT: vector<u8> = b"my const";
```

## 结构体

### 能力要添加 `Cap` 后缀

```move
// bad! if it's a capability, add a `Cap` suffix
public struct Admin has key, store {
    id: UID,
}

// good! reviewer knows what to expect from type
public struct AdminCap has key, store {
    id: UID,
}
```

### 没有任何能力的结构体不用在名字里添加 `Potato` 

```move
// bad! it has no abilities, we already know it's a Hot-Potato type
public struct PromisePotato {}

// good!
public struct Promise {}
```

### Events 命名时要使用过去式

```move
// bad! not clear what this struct does
public struct RegisterUser has copy, drop { user: address }

// good! clear, it's an event
public struct UserRegistered has copy, drop { user: address }
```

### Use Positional Structs for Dynamic Field Keys + `Key` Suffix

```move
// not as bad, but goes against canonical style
public struct DynamicField has copy, drop, store {}

// good! canonical style, Key suffix
public struct DynamicFieldKey() has copy, drop, store;
```

## 函数

### 不要使用 `public entry`, 只用 `public` 或者 `entry`

```move
// bad! entry is not required for a function to be callable in a transaction
public entry fun do_something() { /* ... */ }

// good! public functions are more permissive, can return value
public fun do_something_2(): T { /* ... */ }
```

### 为 PTBs 编写可组合的函数

```move
// bad! not composable, harder to test!
public fun mint_and_transfer(ctx: &mut TxContext) {
    /* ... */
    transfer::transfer(nft, ctx.sender());
}

// good! composable!
public fun mint(ctx: &mut TxContext): NFT { /* ... */ }

// good! intentionally not composable
entry fun mint_and_keep(ctx: &mut TxContext) { /* ... */ }
```

### 对象第一 (Clock除外)

```move
// bad! hard to read!
public fun call_app(
    value: u8,
    app: &mut App,
    is_smth: bool,
    cap: &AppCap,
    clock: &Clock,
    ctx: &mut TxContext,
) { /* ... */ }

// good!
public fun call_app(
    app: &mut App,
    cap: &AppCap,
    value: u8,
    is_smth: bool,
    clock: &Clock,
    ctx: &mut TxContext,
) { /* ... */ }
```

### 能力第二

```move
// bad! breaks method associativity
public fun authorize_action(cap: &AdminCap, app: &mut App) { /* ... */ }

// good! keeps Cap visible in the signature and maintains `.calls()`
public fun authorize_action(app: &mut App, cap: &AdminCap) { /* ... */ }
```

### 使用字段名+ `_mut` 来命名Getters函数

```move
// bad! unnecessary `get_`
public fun get_name(u: &User): String { /* ... */ }

// good! clear that it accesses field `name`
public fun name(u: &User): String { /* ... */ }

// good! for mutable references use `_mut`
public fun details_mut(u: &mut User): &mut Details { /* ... */ }
```

## 函数体: 结构体方法

### 常见的 Coin 操作

```move
// bad! legacy code, hard to read!
let paid = coin::split(&mut payment, amount, ctx);
let balance = coin::into_balance(paid);

// good! struct methods make it easier!
let balance = payment.split(amount, ctx).into_balance();

// even better (in this example - no need to create temporary coin)
let balance = payment.balance_mut().split(amount);

// also can do this!
let coin = balance.into_coin(ctx);
```

### 不需要导入 `std::string::utf8`

```move
// bad! unfortunately, very common!
use std::string::utf8;

let str = utf8(b"hello, world!");

// good!
let str = b"hello, world!".to_string();

// also, for ASCII string
let ascii = b"hello, world!".to_ascii_string();
```

### 直接使用UID 的 `delete`

```move
// bad!
object::delete(id);

// good!
id.delete();
```

### 直接使用`ctx` 的 `sender()`

```move
// bad!
tx_context::sender(ctx);

// good!
ctx.sender()
```

### 使用字面量来创建 Vector，使用Vector的内置函数

```move
// bad!
let mut my_vec = vector::empty();
vector::push_back(&mut my_vec, 10);
let first_el = vector::borrow(&my_vec);
assert!(vector::length(&my_vec) == 1);

// good!
let mut my_vec = vector[10];
let first_el = my_vec[0];
assert!(my_vec.length() == 1);
```

### 集合支持索引语法

```move
let x: VecMap<u8, String> = /* ... */;

// bad!
x.get(&10);
x.get_mut(&10);

// good!
&x[&10];
&mut x[&10];
```

## 可选类型 -> 宏

### 销毁一个可选类型然后以其为参数调用函数

```move
// bad!
if (opt.is_some()) {
    let inner = opt.destroy_some();
    call_function(inner);
};

// good! there's a macro for it!
opt.do!(|value| call_function(value));
```

### 销毁 `Option` 并返回默认值

```move
let opt = option::none();

// bad!
let value = if (opt.is_some()) {
    opt.destroy_some()
} else {
    abort EError
};

// good! there's a macro!
let value = opt.destroy_or!(default_value);

// you can even do abort on `none`
let value = opt.destroy_or!(abort ECannotBeEmpty);
```

## 循环 -> 宏

### 循环 N 次

```move
// bad! hard to read!
let mut i = 0;
while (i < 32) {
    do_action();
    i = i + 1;
};

// good! any uint has this macro!
32u8.do!(|_| do_action());
```

### 创建新数组

```move
// harder to read!
let mut i = 0;
let mut elements = vector[];
while (i < 32) {
    elements.push_back(i);
    i = i + 1;
};

// easy to read!
vector::tabulate!(32, |i| i);
```

### 数组遍历

```move
// bad!
let mut i = 0;
while (i < vec.length()) {
    call_function(&vec[i]);
    i = i + 1;
};

// good!
vec.do_ref!(|e| call_function(e));
```

### 遍历并销毁数组

```move
// bad!
while (!vec.is_empty()) {
    call(vec.pop_back());
};

// good!
vec.destroy!(|e| call(e));
```

### 迭代数组并返回计算结果

```move
// bad!
let mut aggregate = 0;
let mut i = 0;

while (i < source.length()) {
    aggregate = aggregate + source[i];
    i = i + 1;
};

// good!
let aggregate = source.fold!(0, |acc, v| {
    acc + v
});
```

### 数组过滤

> Note: `T: drop` in the `source` vector

```move
// bad!
let mut filtered = [];
let mut i = 0;
while (i < source.length()) {
    if (source[i] > 10) {
        filtered.push_back(source[i]);
    };
    i = i + 1;
};

// good!
let filtered = source.filter!(|e| e > 10);
```

## 其他

### 解包时不需要的值可被完全忽略

```move
// bad! very sparse!
let MyStruct { id, field_1: _, field_2: _, field_3: _ } = value;
id.delete();

// good! 2024 syntax
let MyStruct { id, .. } = value;
id.delete();
```

## 测试

### 把 `#[test]` 和 `#[expected_failure(...)]`放在一起

```move
// bad!
#[test]
#[expected_failure]
fun value_passes_check() {
    abort
}

// good!
#[test, expected_failure]
fun value_passes_check() {
    abort
}
```

### 不需清理 `expected_failure` 测试用例

```move
// bad! clean up is not necessary
#[test, expected_failure(abort_code = my_app::EIncorrectValue)]
fun try_take_missing_object_fail() {
    let mut test = test_scenario::begin(@0);
    my_app::call_function(test.ctx());
    test.end();
}

// good! easy to see where test is expected to fail
#[test, expected_failure(abort_code = my_app::EIncorrectValue)]
fun try_take_missing_object_fail() {
    let mut test = test_scenario::begin(@0);
    my_app::call_function(test.ctx());

    abort // will differ from EIncorrectValue
}
```

### 不要在测试模块中以 `test_` 作为测试的前缀

```move
// bad! the module is already called _tests
module my_package::my_module_tests;

#[test]
fun test_this_feature() { /* ... */ }

// good! better function name as the result
#[test]
fun this_feature_works() { /* ... */ }
```

### 非必要时不要使用 `TestScenario`

```move
// bad! no need, only using ctx
let mut test = test_scenario::begin(@0);
let nft = app::mint(test.ctx());
app::destroy(nft);
test.end();

// good! there's a dummy context for simple cases
let ctx = &mut tx_context::dummy();
app::mint(ctx).destroy();
```

### 不要在 `assert!` 中使用终止代码

```move
// bad! may match application error codes by accident
assert!(is_success, 0);

// good!
assert!(is_success);
```

### 可以的话使用 `assert_eq!` 

```move
// bad! old-style code
assert!(result == b"expected_value", 0);

// good! will print both values if fails
use std::unit_test::assert_eq;

assert_eq!(result, expected_value);
```

### 使用黑洞模式来销毁函数

```move
// bad!
nft.destroy_for_testing();
app.destroy_for_testing();

// good! - no need to define special functions for cleanup
use sui::test_utils::destroy;

destroy(nft);
destroy(app);
```

## 注释

### 文档注释以 `///` 开头

```move
// bad! tooling does't support JavaDoc-style comments
/**
 * Cool method
 * @param ...
 */
public fun do_something() { /* ... */ }

// good! will be rendered as a doc comment in docgen and IDE's
/// Cool method!
public fun do_something() { /* ... */ }
```

### 复杂逻辑评论以 `//` 开头

更有利于帮助审核人员理解代码！

```move
// good!
// Note: can underflow if a value is smaller than 10.
// TODO: add an `assert!` here
let value = external_call(value, ctx);
```
