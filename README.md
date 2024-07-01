# The Move Book (中文版)

本仓库提供 [the Move Book](https://move-book.com) 和 [Move Language Reference](https://move-book.com/reference) 的中文翻译.

原版仓库：[move-book](https://github.com/MystenLabs/move-book)

## 仓库结构

- `the Move Book` 和 `Move Language Reference` 的原版文档分别位于 `book` 和 `reference` 目录。`book` 目录包含主要的书籍内容，而 `reference` 目录包含参考书内容。
- 两份文档的中文翻译分别位于 `book_zh` 和 `reference_zh` 目录。。
- `theme` 目录与两本书相关联，包含主题文件、字体和样式。
- `packages` 目录包含两本书中使用的代码示例。

## 本地运行

您需要安装 [mdBook](https://rust-lang.github.io/mdBook/guide/installation.html) 以便本地运行。

安装完毕，运行以下命令即可本地运行对应文档：

```bash
$ mdbook serve book
$ mdbook serve reference

$ mdbook serve book_zh
$ mdbook serve reference_zh
```

*通过 http://localhost:3000 即可访问。*

## 归档

旧版本文档请查看 `archive` 分支。
