# 安装 Sui

Move 是一种编译型语言，因此你需要安装一个编译器来编写和运行 Move 程序。编译器包含在 Sui 二进制文件中，可以通过以下方法之一进行安装或下载。

## 下载二进制文件

你可以从 [发布页面](https://github.com/MystenLabs/sui/releases) 下载最新的 Sui 二进制文件。该二进制文件适用于 macOS、Linux 和 Windows。对于教育目的和开发，我们推荐使用 `mainnet` 版本。

## 使用 Homebrew 安装 (MacOS)

你可以使用 [Homebrew](https://brew.sh/) 包管理器安装 Sui。

```bash
brew install sui
```

## 使用 Chocolatey 安装 (Windows)

你可以使用 Windows 的 [Chocolatey](https://chocolatey.org/install) 包管理器安装 Sui。

```bash
choco install sui
```

## 使用 Cargo 构建安装 (MacOS, Linux)

你可以使用 Cargo 包管理器本地安装和构建 Sui（需要 Rust）

```bash
cargo install --git https://github.com/MystenLabs/sui.git --bin sui --branch mainnet
```

确保你的系统有最新版本的 Rust，可以使用以下命令更新。

```bash
rustup update stable
```

## 故障排除

有关安装过程的故障排除，请参考 [安装 Sui](https://docs.sui.io/guides/developer/getting-started/sui-install) 指南。