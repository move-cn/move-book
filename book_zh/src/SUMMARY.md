# Summary

<!--

    Things that we don't have:
        - VM and bytecode
        - borrow checker

    Thoughts:
        - someone will jump, some sections will be skipped, some will be read in a different order;

    Audiences:
        - people who don't know anything about Move
        - people who know Move but don't know Sui
        - people who know Sui but don't know Move
        - people who tried Move and Sui and need more

 -->

<!--

- wrapped objects ???
- gas considerations
- custom transfer rules
- object and package versioning

-->

<!-- - [The Move Book](README.md) -->

- [关于本书](README.md)
- [前言](foreword.md)
<!-- - [Introduction](introduction.md) -->
- [开始](before-we-begin/README.md)
  - [安装 Sui](before-we-begin/install-sui.md)
  - [设置 IDE](before-we-begin/ide-support.md)
  - [Move 2024](before-we-begin/move-2024.md)
- [你好 Move !](your-first-move/hello-world.md)
- [你好 Sui !](your-first-move/hello-sui.md)
<!--
    - [Prepare Package]()
    - [Create Account]()
    - [Publishing]()
    - [Sending Transactions]()
    - [Code Walkthrough]()
    - [Ideas]()
    - [Debugging]()
    - [Generating Docs]()
-->
- [概念](./concepts/README.md)
  - [包](./concepts/packages.md)
  - [Manifest](./concepts/manifest.md)
  - [地址](./concepts/address.md)
  - [账户](./concepts/what-is-an-account.md)
  - [交易](./concepts/what-is-a-transaction.md)
- [Move 基础](./move-basics/README.md)
  - [模块](./move-basics/module.md)
  - [注释](./move-basics/comments.md)
  - [原始类型](./move-basics/primitive-types.md)
  - [地址类型](./move-basics/address.md)
  - [表达式](./move-basics/expression.md)
  - [结构体](./move-basics/struct.md)
  - [能力: 介绍](./move-basics/abilities-introduction.md)
  - [能力: Drop](./move-basics/drop-ability.md)
  - [导入模块](./move-basics/importing-modules.md)
  - [标准库](./move-basics/standard-library.md)
  - [数组](./move-basics/vector.md)
  - [可选类型](./move-basics/option.md)
  - [字符串](./move-basics/string.md)
  - [条件控制](./move-basics/control-flow.md)
  - [常量](./move-basics/constants.md)
  - [中断运行](./move-basics/assert-and-abort.md)
  - [方法](./move-basics/function.md)
  - [结构体方法](./move-basics/struct-methods.md)
  - [可见修饰符](./move-basics/visibility.md)
  - [所有权范围](./move-basics/ownership-and-scope.md)
  - [能力: Copy](./move-basics/copy-ability.md)
  - [References](./move-basics/references.md)
  - [泛型](./move-basics/generics.md)
  - [类型反射](./move-basics/type-reflection.md)
  - [测试](./move-basics/testing.md)
- [对象模型](./object/README.md) -pattern
  - [数字资产语言](./object/digital-assets.md)
  - [演进Move](./object/evolution-of-move.md)
  - [对象](./object/object-model.md)
  - [所有权](./object/ownership.md)
  - [路径 & 共识](./object/fast-path-and-consensus.md)
- [使用对象](./storage/README.md)
  - [能力: Key](./storage/key-ability.md)
  - [存储方法](./storage/storage-functions.md)
    <!-- - [Prices and Rebates]() -->
  - [能力: Store](./storage/store-ability.md)
  - [UID and ID](./storage/uid-and-id.md)
  - [限制和转移](./storage/transfer-restrictions.md)
  - [Transfer to Object?]() <!-- (./storage/transfer-to-object.md) -->
- [高级部分](./programmability/README.md)
  - [事务上下文](./programmability/transaction-context.md)
  - [模块初始化](./programmability/module-initializer.md)
  - [设计模式: 能力](./programmability/capability.md)
  - [纪元和时间](./programmability/epoch-and-time.md)
  - [集合](./programmability/collections.md)
  - [动态字段](./programmability/dynamic-fields.md)
  - [动态对象字段](./programmability/dynamic-object-fields.md)
  - [动态集合](./programmability/dynamic-collections.md)
  - [设计模式: 见证者](./programmability/witness-pattern.md)
  - [一次性见证者](./programmability/one-time-witness.md)
  - [发布者权限](./programmability/publisher.md)
  - [Display 对象](./programmability/display.md) <!-- End Block: from Witness to Display -->
  - [事件](./programmability/events.md)
  - [Balance & Coin]() <!-- ./programmability/balance-and-coin.md) -->
  - [Sui Framework](./programmability/sui-framework.md)
  - [设计模式: 烫手山芋](./programmability/hot-potato-pattern.md) <!-- ./programmability/hot-potato.md) -->
  - [设计模式: 请求]()
  - [设计模式: 对象能力]()
  - [升级]()<!-- (./programmability/package-upgrades.md) -->
  - [交易块]()<!-- (./programmability/transaction-blocks.md) -->
  - [Authorization Patterns]()<!-- (./programmability/authorization-patterns.md) -->
  - [加密和哈希]()<!-- (./programmability/cryptography-and-hashing.md) -->
  - [随机数]()<!-- (./programmability/randomness.md) -->
  - [BCS](./programmability/bcs.md)

# Guides

- [2024合并指南](./guides/2024-migration-guide.md)
- [升级实践](./guides/upgradeability-practices.md)
- [限制](./guides/building-against-limits.md)
- [错误处理](./guides/better-error-handling.md)
- [代码质量检查清单](./guides/code-quality-checklist.md)
- [Open-sourcing Libraries]()
- [Creating an NFT Collection]()
- [Tests with Objects]()<!-- (./guides/testing.md) -->
<!-- - [Debugging]()(./guides/debugging.md) -->
- [编码规范](./guides/coding-conventions.md)

# Appendix

- [A - 术语](./appendix/glossary.md)
- [B - 预留地址](./appendix/reserved-addresses.md)
- [C - 出版物](./appendix/publications.md)
- [D - 贡献者](./appendix/contributing.md)
- [E - 致谢](./appendix/acknowledgements.md)
