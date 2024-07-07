---
id: move-book
title: Move Book
custom_edit_url: https://github.com/move-language/move/edit/main/language/documentation/book/README.md
---

为了更新 Move 书籍并预览更改，您需要安装 [`mdbook`](https://rust-lang.github.io/mdBook/guide/installation.html)。您可以通过 `cargo install mdbook` 安装它。

安装完 mdbook 后，您可以通过 `mdbook serve` 预览更改，或者如果希望在对书籍进行更改时输出也随之改变，可以使用 `mdbook watch`。更多选项的信息可以在 [mdbook 网站](https://rust-lang.github.io/mdBook/) 上找到。

完成对 Move 书籍的更改后，您可以创建一个 PR 来更新 Move 书籍网站。以下是过去使用过的流程，已知是有效的，但可能会有更好的方法：

1. 在此目录中运行 `mdbook build`。这将创建一个名为 `book` 的目录。将其复制到 Move git 树外的某个位置 `L`。

2. 确保您的上游仓库是最新的，并切换到 `upstream/gh-pages` 分支。

3. 一旦切换到了 `upstream/gh-pages`，确保您位于仓库的根目录。您应该看到一些 `.html` 文件。现在可以将目录 `L` 中的所有内容移动到此位置：`mv L/* .`

4. 完成后检查一切是否符合您的预期。如果一切正常，请提交一个 PR 到 Move 主仓库的 **`gh-pages`** 分支。

完成后，按照正常的 PR 流程进行。一旦 PR 被接受并合并，更新后的 Move 书籍应该会立即显示在网站上。

**注意：** 当向书籍添加新的（子）章节时，**必须**在 `SUMMARY.md` 文件中包含新文件，否则它不会出现在更新后的书籍中。