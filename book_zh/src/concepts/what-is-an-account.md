# 账户

账户 (account) 是识别用户的一种方式。账户由私钥生成，并通过地址来识别。账户可以拥有对象，并且可以发送交易。每个交易都有一个发送者，发送者通过[地址](./address.md)来识别。

Sui 支持多种加密算法用于生成账户。支持的曲线有 ed25519、secp256k1，还有一种特殊的账户生成方式 - zklogin。Sui 的加密灵活性使得账户生成具有灵活性和多样性。

## 进一步阅读

- [Cryptography in Sui ](https://blog.sui.io/wallet-cryptography-specifications/) - 来自[Sui 博客](https://blog.sui.io)
- [Keys and Addresses](https://docs.sui.io/concepts/cryptography/transaction-auth/keys-addresses) - 来自[Sui 文档](https://docs.sui.io)
- [Signatures](https://docs.sui.io/concepts/cryptography/transaction-auth/signatures) - 来自[Sui 文档](https://docs.sui.io)
