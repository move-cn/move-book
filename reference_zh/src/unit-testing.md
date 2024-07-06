# Unit Tests（单元测试）

Move 的单元测试在 Move 源语言中使用三个注解：

- `#[test]` 标记一个函数为测试；
- `#[expected_failure]` 标记一个测试预期会失败；
- `#[test_only]` 标记一个模块或模块成员（[`use`](./uses.md)、[函数](./functions.md)、[结构体](./structs.md)或[常量](./constants.md)）仅用于测试。

这些注解可以放置在任何适当的形式上，并具有任何可见性。每当一个模块或模块成员被注解为 `#[test_only]` 或 `#[test]` 时，除非为测试编译，否则它不会包含在已编译的字节码中。

## 测试注解

`#[test]` 注解只能放在没有参数的函数上。这个注解将函数标记为由单元测试框架运行的测试。

```move
#[test] // 正确
fun this_is_a_test() { ... }

#[test] // 编译失败，因为测试接受了一个参数
fun this_is_not_correct(arg: u64) { ... }
```

测试还可以标注为 `#[expected_failure]`。这个注解标记测试预期会引发错误。有许多选项可以与 `#[expected_failure]` 注解一起使用，以确保只有在指定条件下失败才会被标记为通过，这些选项在[预期失败](#expected-failures)部分详细介绍。只有具有 `#[test]` 注解的函数也可以被注解为 `#[expected_failure]`。

以下是一些使用 `#[expected_failure]` 注解的简单示例：

```move
#[test]
#[expected_failure]
public fun this_test_will_abort_and_pass() { abort 1 }

#[test]
#[expected_failure]
public fun test_will_error_and_pass() { 1/0; }

#[test] // 通过，因为测试失败并且返回预期的中止代码常量。
#[expected_failure(abort_code = ENotFound)] // ENotFound 是模块中定义的常量
public fun test_will_error_and_pass_abort_code() { abort ENotFound }

#[test] // 失败，因为测试失败并返回一个不同于预期的错误。
#[expected_failure(abort_code = my_module::ENotFound)]
public fun test_will_error_and_fail() { 1/0; }

#[test, expected_failure] // 可以在一个属性中有多个注解。此测试将通过。
public fun this_other_test_will_abort_and_pass() { abort 1 }
```

> **注意**: `#[test]` 和 `#[test_only]` 函数也可以调用 [`entry`](./functions.md#entry-modifier) 函数，无论其可见性如何。

## 预期失败

可以使用 `#[expected_failure]` 注解来指定不同类型的错误条件。包括以下几种方式：

### 1. `#[expected_failure(abort_code = <constant>)]`

如果测试中止并且返回的常量值与模块中定义的常量相同，则测试通过，否则测试失败。这是测试预期失败的推荐方式。

> **注意**: 你可以在 `expected_failure` 注解中引用当前模块或包之外的常量。

```move
module pkg_addr::other_module {
    const ENotFound: u64 = 1;
    public fun will_abort() {
        abort ENotFound
    }
}

module pkg_addr::my_module {
    use pkg_addr::other_module;
    const ENotFound: u64 = 1;

    #[test]
    #[expected_failure(abort_code = ENotFound)]
    fun test_will_abort_and_pass() { abort ENotFound }

    #[test]
    #[expected_failure(abort_code = other_module::ENotFound)]
    fun test_will_abort_and_pass() { other_module::will_abort() }

    // 失败：因为我们期望的常量来自错误的模块。
    #[test]
    #[expected_failure(abort_code = ENotFound)]
    fun test_will_abort_and_pass() { other_module::will_abort() }
}
```

### 2. `#[expected_failure(arithmetic_error, location = <location>)]`

这指定了测试预期会在指定位置失败，并引发算术错误（例如，整数溢出，除以零等）。`<location>` 必须是模块位置的有效路径，例如 `Self` 或 `my_package::my_module`。

```move
module pkg_addr::other_module {
    public fun will_arith_error() { 1/0; }
}

module pkg_addr::my_module {
    use pkg_addr::other_module;
    #[test]
    #[expected_failure(arithmetic_error, location = Self)]
    fun test_will_arith_error_and_pass1() { 1/0; }

    #[test]
    #[expected_failure(arithmetic_error, location = pkg_addr::other_module)]
    fun test_will_arith_error_and_pass2() { other_module::will_arith_error() }

    // 失败：因为我们期望它失败的位置与测试实际失败的位置不同。
    #[test]
    #[expected_failure(arithmetic_error, location = Self)]
    fun test_will_arith_error_and_fail() { other_module::will_arith_error() }
}
```

### 3. `#[expected_failure(out_of_gas, location = <location>)]`

这指定了测试预期会在指定位置失败，并引发耗尽气体错误。`<location>` 必须是模块位置的有效路径，例如 `Self` 或 `my_package::my_module`。

```move
module pkg_addr::other_module {
    public fun will_oog() { loop {} }
}

module pkg_addr::my_module {
    use pkg_addr::other_module;
    #[test]
    #[expected_failure(out_of_gas, location = Self)]
    fun test_will_oog_and_pass1() { loop {} }

    #[test]
    #[expected_failure(arithmetic_error, location = pkg_addr::other_module)]
    fun test_will_oog_and_pass2() { other_module::will_oog() }

    // 失败：因为我们期望它失败的位置与测试实际失败的位置不同。
    #[test]
    #[expected_failure(out_of_gas, location = Self)]
    fun test_will_oog_and_fail() { other_module::will_oog() }
}
```

### 4. `#[expected_failure(vector_error, minor_status = <u64_opt>, location = <location>)]`

这指定了测试预期会在指定位置失败，并引发向量错误，并带有给定的 `minor_status`（如果提供）。`<location>` 必须是模块位置的有效路径，例如 `Self` 或 `my_package::my_module`。`<u64_opt>` 是一个可选参数，指定向量错误的次要状态。如果未指定，则测试失败时，只要引发任何次要状态的向量错误，测试就会通过。如果指定了，则只有在测试失败并引发指定次要状态的向量错误时，测试才会通过。

```move
module pkg_addr::other_module {
    public fun vector_borrow_empty() {
        &vector<u64>[][1];
    }
}

module pkg_addr::my_module {
    #[test]
    #[expected_failure(vector_error, location = Self)]
    fun vector_abort_same_module() {
        vector::borrow(&vector<u64>[], 1);
    }

    #[test]
    #[expected_failure(vector_error, location = pkg_addr::other_module)]
    fun vector_abort_same_module() {
        other_module::vector_borrow_empty();
    }

    // Can specify minor statues (i.e., vector-specific error codes) to expect.
    #[test]
    #[expected_failure(vector_error, minor_status = 1, location = Self)]
    fun native_abort_good_right_code() {
        vector::borrow(&vector<u64>[], 1);
    }

    // FAIL: correct error, but wrong location.
    #[test]
    #[expected_failure(vector_error, location = pkg_addr::other_module)]
    fun vector_abort_same_module() {
        other_module::vector_borrow_empty();
    }

    // FAIL: correct error and location but the minor status differs so this test will fail.
    #[test]
    #[expected_failure(vector_error, minor_status = 0, location = Self)]
    fun vector_abort_wrong_minor_code() {
        vector::borrow(&vector<u64>[], 1);
    }
}
```
### 5. `#[expected_failure]`

这个注解表示，如果测试因_任何_错误代码中止，测试将通过。使用此注解来标注预期失败的测试时应当**极其谨慎**，并始终优先使用上述描述的方式。以下是一些此类注解的示例：

```move
#[test]
#[expected_failure]
fun test_will_abort_and_pass1() { abort 1 }

#[test]
#[expected_failure]
fun test_will_arith_error_and_pass2() { 1/0; }
```

## 仅测试注解

模块及其成员可以声明为仅用于测试。如果一个项目被注解为 `#[test_only]`，则该项目仅在测试模式下编译时才会包含在已编译的 Move 字节码中。此外，在非测试模式下编译时，任何非测试 `use` 的 `#[test_only]` 模块将在编译期间引发错误。

> **注意**: 注解为 `#[test_only]` 的函数仅可从测试代码中调用，但它们本身不是测试，且不会由单元测试框架运行。

```move
#[test_only] // 仅测试属性可以附加到模块
module abc { ... }

#[test_only] // 仅测试属性可以附加到常量
const MY_ADDR: address = @0x1;

#[test_only] // 仅测试属性可以附加到 use
use pkg_addr::some_other_module;

#[test_only] // 仅测试属性可以附加到结构体
public struct SomeStruct { ... }

#[test_only] // 仅测试属性可以附加到函数。只能从测试代码中调用，但这不是一个测试！
fun test_only_function(...) { ... }
```

## 运行单元测试

Move 包的单元测试可以使用 [`sui move test` 命令](./packages.md)运行。

在运行测试时，每个测试将 `PASS`、`FAIL` 或 `TIMEOUT`。如果测试用例失败，如果可能，将报告失败的位置和引起失败的函数名称。可以在下面的示例中看到这一点。

如果测试用例超出了可以为任何单个测试执行的最大指令数，则该测试将被标记为超时。可以使用以下选项更改此界限。此外，虽然测试结果始终是确定性的，但默认情况下测试是并行运行的，因此除非使用单线程运行，否则测试结果的顺序是非确定性的，这可以通过一个选项进行配置。

上述选项是许多选项中的两个，可以微调测试并帮助调试失败的测试。要查看所有可用选项及其描述，请将 `--help` 标志传递给 `sui move test` 命令：

```
$ sui move test --help
```

## 示例

以下示例展示了一个使用一些单元测试功能的简单模块：

首先创建一个空包并切换到该目录：

```bash
$ sui move new test_example; cd test_example
```

然后在 `sources` 目录下添加以下模块：

```move
// 文件名：sources/my_module.move
module test_example::my_module {

    public struct Wrapper(u64)

    const ECoinIsZero: u64 = 0;

    public fun make_sure_non_zero_coin(coin: Wrapper): Wrapper {
        assert!(coin.0 > 0, ECoinIsZero);
        coin
    }

    #[test]
    fun make_sure_non_zero_coin_passes() {
        let coin = Wrapper(1);
        let Wrapper(_) = make_sure_non_zero_coin(coin);
    }

    #[test]
    // 或者 #[expected_failure] 如果我们不关心中止代码
    #[expected_failure(abort_code = ECoinIsZero)]
    fun make_sure_zero_coin_fails() {
        let coin = Wrapper(0);
        let Wrapper(_) = make_sure_non_zero_coin(coin);
    }

    #[test_only] // 仅用于测试的辅助函数
    fun make_coin_zero(coin: &mut Wrapper) {
        coin.0 = 0;
    }

    #[test]
    #[expected_failure(abort_code = ECoinIsZero)]
    fun make_sure_zero_coin_fails2() {
        let mut coin = Wrapper(10);
        make_coin_zero(&mut coin);
        let Wrapper(_) = make_sure_non_zero_coin(coin);
    }
}
```

### 运行测试

你可以使用 `move test` 命令运行这些测试：

```bash
$ sui move test
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING test_example
Running Move unit tests
[ PASS    ] 0x0::my_module::make_sure_non_zero_coin_passes
[ PASS    ] 0x0::my_module::make_sure_zero_coin_fails
[ PASS    ] 0x0::my_module::make_sure_zero_coin_fails2
Test result: OK. Total tests: 3; passed: 3; failed: 0
```

### 使用测试标志

#### 运行特定测试

你可以使用 `sui move test <str>` 运行特定测试或一组测试。这将只运行名称中包含 `<str>` 的测试。例如，如果我们只想运行名称中包含 `"non_zero"` 的测试：

```bash
$ sui move test non_zero
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING test_example
Running Move unit tests
[ PASS    ] 0x0::my_module::make_sure_non_zero_coin_passes
Test result: OK. Total tests: 1; passed: 1; failed: 0
```

#### `-i <bound>` 或 `--gas_used <bound>`

这将限定任何一个测试可以消耗的最大气体量为 `<bound>`：

```bash
$ sui move test -i 0
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING test_example
Running Move unit tests
[ TIMEOUT ] 0x0::my_module::make_sure_non_zero_coin_passes
[ FAIL    ] 0x0::my_module::make_sure_zero_coin_fails
[ FAIL    ] 0x0::my_module::make_sure_zero_coin_fails2

Test failures:

Failures in 0x0::my_module:

┌── make_sure_non_zero_coin_passes ──────
│ Test timed out
└──────────────────


┌── make_sure_zero_coin_fails ──────
│ error[E11001]: test failure
│    ┌─ ./sources/my_module.move:22:27
│    │
│ 21 │     fun make_sure_zero_coin_fails() {
│    │         ------------------------- In this function in 0x0::my_module
│ 22 │         let coin = MyCoin(0);
│    │                           ^ Test did not error as expected. Expected test to abort with code 0 <SNIP>
│
│
└──────────────────


┌── make_sure_zero_coin_fails2 ──────
│ error[E11001]: test failure
│    ┌─ ./sources/my_module.move:34:31
│    │
│ 33 │     fun make_sure_zero_coin_fails2() {
│    │         -------------------------- In this function in 0x0::my_module
│ 34 │         let mut coin = MyCoin(10);
│    │                               ^^ Test did not error as expected. Expected test to abort with code 0 <SNIP>
│
│
└──────────────────

Test result: FAILED. Total tests: 3; passed: 0; failed: 3
```

#### `-s` or `--statistics`

使用这些标志，您可以收集运行的测试的统计信息，并报告每个测试的运行时间和气体使用量。您还可以添加 `csv` (`sui move test -s csv`) 以获取气体使用量的 CSV 输出格式。例如，如果我们想查看上述示例中测试的统计信息：

```bash
$ sui move test -s
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING test_example
Running Move unit tests
[ PASS    ] 0x0::my_module::make_sure_non_zero_coin_passes
[ PASS    ] 0x0::my_module::make_sure_zero_coin_fails
[ PASS    ] 0x0::my_module::make_sure_zero_coin_fails2

Test Statistics:

┌────────────────────────────────────────────────┬────────────┬───────────────────────────┐
│                   Test Name                    │    Time    │         Gas Used          │
├────────────────────────────────────────────────┼────────────┼───────────────────────────┤
│ 0x0::my_module::make_sure_non_zero_coin_passes │   0.001    │             1             │
├────────────────────────────────────────────────┼────────────┼───────────────────────────┤
│ 0x0::my_module::make_sure_zero_coin_fails      │   0.001    │             1             │
├────────────────────────────────────────────────┼────────────┼───────────────────────────┤
│ 0x0::my_module::make_sure_zero_coin_fails2     │   0.001    │             1             │
└────────────────────────────────────────────────┴────────────┴───────────────────────────┘

Test result: OK. Total tests: 3; passed: 3; failed: 0
```
