# Manifest

`Move.toml` 是描述 [包](./packages.md) 及其依赖关系的清单文件，采用 [TOML](https://toml.io/en/) 格式，包含多个部分，其中最重要的是 `[package]`、`[dependencies]` 和 `[addresses]`。

```toml
[package]
name = "my_project"
version = "0.0.0"
edition = "2024"

[dependencies]
Sui = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }

[addresses]
std =  "0x1"
alice = "0xA11CE"

[dev-addresses]
alice = "0xB0B"
```

## 各部分详解

### Package

`[package]` 部分用于描述包。该部分的字段不会被发布到链上，但会用于工具和版本管理；它还指定了编译器使用的 Move 版本。

- `name` - 导入时包的名称；
- `version` - 包的版本，可用于版本管理；
- `edition` - Move 语言的版本；目前唯一有效的值是 `2024`。

### Dependencies

`[dependencies]` 部分用于指定项目的依赖关系。每个依赖关系都以键值对的形式指定，键是依赖的名称，值是依赖的规范。依赖规范可以是 git 仓库的 URL 或本地目录的路径。

```toml
# git 仓库
Sui = { git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }

# 本地目录
MyPackage = { local = "../my-package" }
```

包还可以从其他包导入地址。例如，Sui 依赖项将 `std` 和 `sui` 地址添加到项目中。这些地址可以在代码中用作地址的别名。

### 使用 override 解决版本冲突

有时候依赖项会有相同包的不同版本之间的冲突。例如，如果有两个依赖项使用了不同版本的 Sui 包，可以在 `[dependencies]` 部分使用 `override` 字段来解决冲突。在依赖关系中指定的版本将覆盖依赖项本身指定的版本。

```toml
[dependencies]
Sui = { override = true, git = "https://github.com/MystenLabs/sui.git", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }
```

### Dev-dependencies

可以在清单中添加 `[dev-dependencies]` 部分，用于在开发和测试模式下覆盖依赖关系。例如，如果想在开发模式下使用不同版本的 Sui 包，可以在 `[dev-dependencies]` 部分添加自定义的依赖规范。

### Addresses

`[addresses]` 部分用于为地址添加别名。可以在此部分指定任何地址，然后在代码中作为别名使用。例如，如果将 `alice = "0xA11CE"` 添加到此部分，可以在代码中使用 `alice` 代替 `0xA11CE`。

### Dev-addresses

`[dev-addresses]` 部分与 `[addresses]` 类似，但仅在测试和开发模式下有效。需要注意的是，这部分无法引入新的别名，只能覆盖现有的别名。因此，在上面的示例中，如果将 `alice = "0xB0B"` 添加到此部分，那么在测试和开发模式下，`alice` 地址将是 `0xB0B`，而在常规构建中则是 `0xA11CE`。

## TOML 样式

TOML 格式支持两种表格样式：内联样式和多行样式。上面的示例使用的是内联样式，但对于依赖关系，多行样式也是可行的。

```toml
# 内联样式
[dependencies]
Sui = { override = true, git = "", subdir = "crates/sui-framework/packages/sui-framework", rev = "framework/testnet" }
MyPackage = { local = "../my-package" }
```

```toml
# 多行样式
[dependencies.Sui]
override = true
git = "https://github.com/MystenLabs/sui.git"
subdir = "crates/sui-framework/packages/sui-framework"
rev = "framework/testnet"

[dependencies.MyPackage]
local = "../my-package"
```

## 进一步阅读

- Move Reference 中的 [Packages](/reference/packages.html) 章节