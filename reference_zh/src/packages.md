# Packages（包）

包允许Move程序员更轻松地重用代码，并在项目之间共享代码。Move包系统使程序员能够轻松地：

- 定义包含Move代码的包；
- 通过[命名地址](./primitive-types/address.md)对包进行参数化；
- 在其他Move代码中导入和使用包，并实例化命名地址；
- 构建包并生成与包相关的编译工件；
- 在编译后与编译的Move工件周围使用公共接口。

## 包布局和清单语法

Move包源目录包含一个 `Move.toml` 包清单文件，一个生成的 `Move.lock` 文件，以及一组子目录：

```plaintext
a_move_package
├── Move.toml      (必需)
├── Move.lock      (生成的)
├── sources        (必需)
├── doc_templates  (可选)
├── examples       (可选，测试和开发模式)
└── tests          (可选，测试模式)
```

标记为 "必需" 的目录和文件必须存在，才能被视为一个Move包并进行构建。可选目录可以存在，如果存在，则根据构建包的模式将其包含在编译过程中。例如，在“dev”或“test”模式下构建时，`tests` 和 `examples` 目录也会被包含进来。

依次查看每个内容：

1. [`Move.toml`](#movetoml) 文件是包清单，是一个Move包被视为必需的文件。该文件包含关于包的元数据，如名称、依赖关系等。
2. [`Move.lock`](#movelock) 文件由Move CLI生成，包含了包及其依赖项的固定构建版本。它用于确保在不同的构建中使用一致的版本，并确保依赖项的更改在该文件中体现为变更。
3. `sources` 目录是必需的，包含构成包的Move模块。此目录中的模块将始终包含在编译过程中。
4. `doc_templates` 目录可以包含文档模板，用于生成包的文档。
5. `examples` 目录可以包含额外的代码，仅用于开发和/或教程，这些内容不会在非`test`或`dev`模式下编译。
6. `tests` 目录可以包含仅在`test`模式下编译或运行[Move单元测试](./unit-testing.md)的Move模块。

### Move.toml

Move包清单在 `Move.toml` 文件中定义，具有以下语法。可选字段用 `*` 标记，`+` 表示一个或多个元素：

```ini
[package]
name = <string>
edition* = <string>      # 例如，"2024.alpha" 表示使用Move 2024版，目前处于alpha阶段。如果未指定，则默认为最新的稳定版。
license* = <string>              # 例如，"MIT", "GPL", "Apache 2.0"
authors* = [<string>,+]  # 例如，["Joe Smith (joesmith@noemail.com)", "John Snow (johnsnow@noemail.com)"]

# 外部工具可以向该部分添加额外的字段。例如，在Sui上，会添加以下部分：
published-at* = "<hex-address>" # 包的发布地址。应在第一次发布后设置。

[dependencies] # （可选部分）依赖项的路径
# 一个或多个行声明依赖项，格式如下

# ##### 本地依赖项 #####
# 对于本地依赖项，请使用 `local = path`。路径相对于包根目录
# Local = { local = "../path/to" }
# 若要解决版本冲突并强制指定依赖项的特定版本，可以使用 `override = true`
# Override = { local = "../conflicting/version", override = true }
# 要对依赖项中的地址值进行实例化，请使用 `addr_subst`
<string> = {
    local = <string>,
    override* = <bool>,
    addr_subst* = { (<string> = (<string> | "<hex_address>"))+ }
}

# ##### Git依赖项 #####
# 对于远程导入，请使用 `{ git = "...", subdir = "...", rev = "..." }`。
# 必须提供修订版，可以是分支、标签或提交哈希。
# 如果未指定 `subdir`，则使用存储库的根目录。
# MyRemotePackage = { git = "https://some.remote/host.git", subdir = "remote/path", rev = "main" }
<string> = {
    git = <URL ending in .git>,
    subdir=<path to dir containing Move.toml inside git repo>,
    rev=<git commit hash>,
    override* = <bool>,
    addr_subst* = { (<string> = (<string> | "<hex_address>"))+ }
}

[addresses]  # （可选部分）在此包中声明命名地址
# 一个或多个行声明命名地址，格式如下
# 与包名称匹配的地址必须设置为 `"0x0"`，否则将无法发布。
<addr_name> = "_" | "<hex_address>" # 例如，std = "_" 或 my_addr = "0xC0FFEECAFE"

# 命名地址在Move中可以使用 `@name` 访问。它们也可以被导出：
# 例如，`std = "0x1"` 是标准库导出的。
# alice = "0xA11CE"

[dev-dependencies] # （可选部分）与 [dependencies] 部分相同，但仅在“dev”和“test”模式下包含
# dev-dependencies 部分允许在“--test”和“--dev”模式下覆盖依赖项。例如，您可以在这里引入仅用于测试的依赖项。
# Local = { local = "../path/to/dev-build" }
<string> = {
    local = <string>,
    override* = <bool>,
    addr_subst* = { (<string> = (<string> | "<hex_address>"))+ }
}
<string> = {
    git = <URL ending in .git>,
    subdir=<path to dir containing Move.toml inside git repo>,
    rev=<git commit hash>,
    override* = <bool>,
    addr_subst* = { (<string> = (<string> | "<hex_address>"))+ }
}

[dev-addresses] # （可选部分）与 [addresses] 部分相同，但仅在“dev”和“test”模式下包含
# dev-addresses 部分允许在“--test”和“--dev”模式下覆盖命名地址。
<addr_name> = "<hex_address>" # 例如，alice = "0xB0B"
```

一个最小的包清单示例：

```ini
[package]
name = "AName"
```

一个更标准的包清单示例，还包括了Move标准库，并通过`LocalDep`包将命名地址`std`实例化为地址值`0x1`：

```ini
[package]
name = "AName"
license = "Apache 2.0"

[addresses]
address_to_be_filled_in = "_"
specified_address = "0xB0B"

[dependencies]
# 本地依赖项
LocalDep = { local = "projects/move-awesomeness", addr_subst = { "std" = "0x1" } }
# Git依赖项
MoveStdlib = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/move-stdlib", rev = "framework/mainnet" }

[dev-addresses] # 用于开发该模块时
address_to_be_filled_in = "0x101010101"
```

大多数包清单中的部分都是不言自明的，但命名地址可能有点难以理解，因此我们将更详细地探讨它们在[编译期间的命名地址](#named-addresses-during-compilation)。

## 编译期间的命名地址



回想一下，Move具有[命名地址](./primitive-types/address.md)，而命名地址不能在Move中声明。相反，它们在包级别声明：在Move包的清单文件（`Move.toml`）中，您可以在包中声明命名地址，在Move包系统中实例化其他命名地址，并重命名来自其他包的命名地址。

让我们逐一浏览每个操作，以及它们如何在包的清单中执行：

### 声明命名地址

假设我们在 `example_pkg/sources/A.move` 中有一个Move模块，如下所示：

```move
module named_addr::a {
    public fun x(): address { @named_addr }
}
```

我们可以在 `example_pkg/Move.toml` 中两种不同方式声明命名地址 `named_addr`。首先是：

```ini
[package]
name = "example_pkg"
...
[addresses]
named_addr = "_"
```

将 `named_addr` 声明为 `example_pkg` 包中的命名地址，并且_此地址可以是任何有效的地址值_。特别地，导入包可以选择将命名地址 `named_addr` 的值设为任何它希望的地址。直观地说，您可以将其视为通过命名地址 `named_addr` 参数化包 `example_pkg`，然后可以稍后由导入包实例化。

`named_addr` 也可以声明为：

```ini
[package]
name = "example_pkg"
...
[addresses]
named_addr = "0xCAFE"
```

这表明命名地址 `named_addr` 确切地是 `0xCAFE`，并且不能更改。这对于其他导入包可以使用这个命名地址而无需担心它的确切值非常有用。

有了这两种不同的声明方法，有两种信息关于命名地址可以在包图中流动的方式：

- 前者（"未分配的命名地址"）允许命名地址值从导入位置流向声明位置。
- 后者（"已分配的命名地址"）允许命名地址值从声明位置向上在包图中流向使用位置。

有了这两种在整个包图中流动命名地址信息的方法，理解作用域和重命名规则变得非常重要。

## 命名地址的作用域和重命名

在包 `P` 中，命名地址 `N` 的作用域遵循以下规则：

1. 如果 `P` 自己声明了命名地址 `N`；
2. 或者，`P` 的一个传递依赖包声明了命名地址 `N`，并且在包图中存在从 `P` 到声明 `N` 的包的依赖路径，并且没有对 `N` 进行重命名。

此外，每个包中的命名地址都是公开的。因此，根据上述作用域规则，每个包在被导入时都会带有一组命名地址，例如，如果导入 `example_pkg`，那么 `named_addr` 命名地址也会随之导入到作用域中。因此，如果包 `P` 导入了两个包 `P1` 和 `P2`，它们都声明了命名地址 `N`，那么在包 `P` 中引用 `N` 时会出现问题：到底是来自 `P1` 还是 `P2` 的 "`N`"？为了避免命名地址的来源不明确，我们要求包内所有依赖引入的作用域集合是不重叠的，并且提供一种在导入包时对命名地址进行 _重命名_ 的方式。

在上面的例子中，可以通过以下方式在 `P`、`P1` 和 `P2` 中进行命名地址重命名：

```ini
[package]
name = "P"
...
[dependencies]
P1 = { local = "some_path_to_P1", addr_subst = { "P1N" = "N" } }
P2 = { local = "some_path_to_P2"  }
```

通过这种重命名，`N` 将引用来自 `P2` 的 `N`，而 `P1N` 将引用来自 `P1` 的 `N`：

```move
module N::A {
    public fun x(): address { @P1N }
}
```

需要注意的是，_重命名不是局部的_：一旦在包 `P` 中将命名地址 `N` 重命名为 `N2`，所有导入 `P` 的包将不再看到 `N`，而只能看到 `N2`，除非 `N` 从 `P` 外部重新引入。这就是本节开头规则 (2) 中指定的作用域规则的原因，即 "包图中从 `P` 到 `N` 声明包的依赖路径没有对 `N` 进行重命名"。

### 命名地址的实例化

命名地址可以在包图中的多个位置多次实例化，只要始终使用相同的值。如果同一个命名地址（无论是否重命名）在包图中的不同位置使用了不同的值，则会出现错误。

Move 包只有在所有命名地址都解析为值时才能编译。如果包希望公开一个未实例化的命名地址，这就是 `[dev-addresses]` 部分的部分解决方案。该部分可以为命名地址设置值，但不能引入任何命名地址。此外，只有根包中的 `[dev-addresses]` 会包含在 `dev` 模式中。例如，具有以下清单的根包在 `dev` 模式外部不会编译，因为 `named_addr` 将未实例化：

```ini
[package]
name = "example_pkg"
...
[addresses]
named_addr = "_"

[dev-addresses]
named_addr = "0xC0FFEE"
```

## 使用和构建产物

Move 包系统作为 CLI 的一部分提供了一个命令行选项：`sui move <command> <command_flags>`。除非提供特定路径，否则所有包命令都将在当前封闭的 Move 包中运行。可以通过运行 `sui move --help` 查找 Move CLI 的所有命令和标志列表。

### 构建产物

可以使用 CLI 命令来编译包。这将创建一个 `build` 目录，其中包含与构建相关的产物（包括字节码二进制文件、源代码映射和文档）。`build` 目录的一般布局如下：

```plaintext
a_move_package
├── BuildInfo.yaml
├── bytecode_modules
│   ├── dependencies
│   │   ├── <dep_pkg_name>
│   │   │   └── *.mv
│   │   ...
│   │   └──  <dep_pkg_name>
│   │       └── *.mv
│   ...
│   └── *.mv
├── docs
│   ├── dependencies
│   │   ├── <dep_pkg_name>
│   │   │   └── *.md
│   │   ...
│   │   └──  <dep_pkg_name>
│   │       └── *.md
│   ...
│   └── *.md
├── source_maps
│   ├── dependencies
│   │   ├── <dep_pkg_name>
│   │   │   └── *.mvsm
│   │   ...
│   │   └──  <dep_pkg_name>
│   │       └── *.mvsm
│   ...
│   └── *.mvsm
└── sources
    ...
    └── *.move
    ├── dependencies
    │   ├── <dep_pkg_name>
    │   │   └── *.move
    │   ...
    │   └──  <dep_pkg_name>
    │       └── *.move
    ...
    └── *.move
```
## Move.lock 文件

当 Move 包构建时，会在 Move 包的根目录生成一个 `Move.lock` 文件。`Move.lock` 文件包含有关您的包及其构建配置的信息，并充当 Move 编译器与其他工具（如特定链的命令行界面和第三方包管理器）之间的通信层。

与 `Move.toml` 文件类似，`Move.lock` 文件是一个基于文本的 TOML 文件。但与包清单不同的是，您不应直接编辑 `Move.lock` 文件。工具链中的进程（如 Move 编译器）会访问和编辑文件，以读取并附加相关信息。此外，您也不应将文件从根目录移动，因为它需要与包清单 `Move.toml` 在同一级别。

如果您在包的源代码控制中使用了版本控制，建议将与所需构建或发布的包对应的 `Move.lock` 文件一并提交。这样可以确保每次构建的包都与原始包完全一致，并且构建的变更将以 `Move.lock` 文件的变更形式体现出来。

### `[move]` 部分

这部分包含了锁文件中所需的核心信息：

- 锁文件版本（用于向后兼容性检查和将来版本化锁文件更改）。
- 用于生成此锁文件的 `Move.toml` 文件的 SHA3-256 哈希值。
- 所有依赖项的 `Move.lock` 文件的哈希值。如果没有依赖项，则为空字符串。
- 依赖项列表。

```ini
[move]
version = <string> # 锁文件版本，用于向后兼容性检查。
manifest_digest = <hash> # 用于生成此锁文件的 Move.toml 文件的 SHA3-256 哈希值。
deps_digest = <hash> # 所有依赖项的 Move.lock 文件的 SHA3-256 哈希值。如果没有依赖项，则为空字符串。
dependencies = { (name = <string>)* } # 依赖项列表。如果没有依赖项则不出现。
```

### `[move.package]` 部分

Move 编译器解析每个依赖项后，将依赖项的位置写入 `Move.lock` 文件。如果依赖项无法解析，则编译器不会写入 `Move.lock` 文件，构建将失败。如果所有依赖项解析成功，则 `Move.lock` 文件将以以下格式包含所有包的传递依赖项的位置（本地和远程）：

```ini
# ...

[[move.package]]
name = "A"
source = { git = "https://github.com/b/c.git", subdir = "e/f", rev = "a1b2c3" }

[[move.package]]
name = "B"
source = { local = "../local-dep" }
```

### `[move.toolchain-version]` 部分

如上所述，外部工具可能会向锁文件添加其他字段。例如，Sui 包管理器向锁文件添加了工具链版本信息，可用于链上源代码验证：

```ini
# ...

[move.toolchain-version]
compiler-version = <string> # 用于构建包的 Move 编译器的版本，例如 "1.21.0"
edition = <string> # 用于构建包的 Move 语言版本，例如 "2024.alpha"
flavor = <string> # 用于构建包的 Move 编译器的特定版本，例如 "sui"
```
