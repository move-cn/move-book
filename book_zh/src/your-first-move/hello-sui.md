# 你好，Sui！

在[上一节](./hello-world.md)中，我们创建了一个新的包并演示了创建、构建和测试一个 Move 包的基本流程。在本节中，我们将编写一个简单的应用程序，该应用程序使用存储模型并可以进行交互。为此，我们将创建一个简单的待办事项列表应用程序。

## 创建一个新的包

按照与[Hello, World!](./hello-world.md)相同的流程，我们将创建一个名为 `todo_list` 的新包。

```bash
$ sui move new todo_list
```

## 添加代码

为了加快速度并专注于应用程序逻辑，我们将提供待办事项列表应用程序的代码。将 _sources/todo_list.move_ 文件的内容替换为以下代码：

> 注意：虽然内容一开始可能会显得有些复杂，我们将在接下来的部分逐步解释。现在请专注于手头的任务。

```move
{{#include ../../../packages/todo_list/sources/todo_list.move:all}}
```

## 构建包

为了确保一切操作正确无误，请运行 `sui move build` 命令来构建包。如果一切顺利，你应该看到类似以下的输出：

```bash
$ sui move build
UPDATING GIT DEPENDENCY https://github.com/MystenLabs/sui.git
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING todo_list
```

如果没有错误，那么说明你已经成功构建了该包。如果有错误，请确保：

- 代码复制正确
- 文件名和包名正确

在这个阶段，代码失败的原因不多。但如果仍然遇到问题，请尝试查看包的结构，位置在 [这里](https://github.com/MystenLabs/move-book/tree/main/packages/todo_list)。

## 设置账户

为了发布和与包交互，我们需要设置一个账户。为了简单和演示的目的，我们将使用 _sui devnet_ 环境。

> 如果你已经设置了账户，请跳过此步骤。

如果你是第一次设置账户，需要运行 `sui client` 命令，然后 CLI 将提示你回答多个问题。下面是答案示例，以 `>` 开头：

```bash
$ sui client
Config file ["/path/to/home/.sui/sui_config/client.yaml"] doesn't exist, do you want to connect to a Sui Full node server [y/N]?
> y
Sui Full node server URL (Defaults to Sui Testnet if not specified) :
>
Select key scheme to generate keypair (0 for ed25519, 1 for secp256k1, 2: for secp256r1):
> 0
```

回答完问题后，CLI 将生成一个新的密钥对并保存到配置文件中。现在你可以使用这个账户与网络交互了。

要检查账户设置是否正确，请运行 `sui client active-address` 命令：

```bash
$ sui client active-address
0x....
```

该命令将输出你账户的地址，以 `0x` 开头，后面跟着64个字符。

## 获取测试币

在 _devnet_ 和 _testnet_ 环境中，CLI 提供了一种方式来请求测试币到你的账户，以便你可以与网络进行交互。要请求测试币，请运行 `sui client faucet` 命令：

```bash
$ sui client faucet
Request successful. It can take up to 1 minute to get the coin. Run sui client gas to check your gas coins.
```

稍等片刻后，你可以运行 `sui client balance` 命令来检查硬币是否已发送到你的账户：

```bash
$ sui client balance
╭────────────────────────────────────────╮
│ Balance of coins owned by this address │
├────────────────────────────────────────┤
│ ╭──────────────────────────────────╮   │
│ │ coin  balance (raw)  balance     │   │
│ ├──────────────────────────────────┤   │
│ │ Sui   1000000000    1.00 SUI     │   │
│ ╰──────────────────────────────────╯   │
╰────────────────────────────────────────╯
```

或者，你可以通过运行 `sui client objects` 命令来查询你的账户拥有的对象。实际输出会有所不同，因为对象 ID 是唯一的，摘要也是唯一的，但结构类似：

```bash
$ sui client objects
╭───────────────────────────────────────────────────────────────────────────────────────╮
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  0x4ea1303e4f5e2f65fc3709bc0fb70a3035fdd2d53dbcff33e026a50a742ce0de  │ │
│ │ version    │  4                                                                   │ │
│ │ digest     │  nA68oa8gab/CdIRw+240wze8u0P+sRe4vcisbENcR4U=                        │ │
│ │ objectType │  0x0000..0002::coin::Coin                                            │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
╰───────────────────────────────────────────────────────────────────────────────────────╯
```

现在我们已经设置了账户并且账户中有了硬币，我们可以开始与网络进行交互。我们将从将包发布到网络开始。

## 发布

要将包发布到网络上，我们将使用 `sui client publish` 命令。该命令将自动构建包，并使用其字节码在单个事务中进行发布。

> 在发布过程中，我们使用 `--gas-budget` 参数指定了事务的 gas 预算。本节不涉及详细讨论这个主题，但重要的是要知道，在 Sui 中，每个交易都需要支付 gas 费用，而 gas 费用是用 SUI 币支付的。

`gas-budget` 以 _MISTs_ 表示。1 SUI 等于 10^9 MISTs。为了演示，我们将使用 100,000,000 MISTs，相当于 0.1 SUI。

```bash
# 在 `todo_list` 文件夹中运行以下命令
$ sui client publish --gas-budget 100000000

# 或者，你可以指定包的路径
$ sui client publish --gas-budget 100000000 todo_list
```

发布命令的输出相对较长，因此我们将分部分展示并解释它。

```bash
$ sui client publish --gas-budget 100000000
UPDATING GIT DEPENDENCY https://github.com/MystenLabs/sui.git
INCLUDING DEPENDENCY Sui
INCLUDING DEPENDENCY MoveStdlib
BUILDING todo_list
Successfully verified dependencies on-chain against source.
Transaction Digest: GpcDV6JjjGQMRwHpEz582qsd5MpCYgSwrDAq1JXcpFjW
```

正如你所见，当我们运行 `publish` 命令时，CLI 首先构建包，然后验证链上的依赖项，最后发布包。命令的输出是事务摘要，这是交易的唯一标识符，可用于查询交易状态。

### 事务数据 (Transaction Data)

`TransactionData` 部分包含我们刚发送的交易信息。它包括字段如 `sender`（发送者地址）、使用 `--gas-budget` 参数设置的 `gas_budget`（gas 预算）以及我们用于支付的币种。它还打印了 CLI 运行的命令。在本示例中，运行了 `Publish` 和 `TransferObject` 命令 - 后者将一个特殊对象 `UpgradeCap` 转移给了发送者。

```plaintext
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Data                                                                                             │
├──────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                                   │
│ Gas Owner: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                                │
│ Gas Budget: 100000000 MIST                                                                                   │
│ Gas Price: 1000 MIST                                                                                         │
│ Gas Payment:                                                                                                 │
│  ┌──                                                                                                         │
│  │ ID: 0x4ea1303e4f5e2f65fc3709bc0fb70a3035fdd2d53dbcff33e026a50a742ce0de                                    │
│  │ Version: 7                                                                                                │
│  │ Digest: AXYPnups8A5J6pkvLa6RekX2ye3qur66EZ88mEbaUDQ1                                                      │
│  └──                                                                                                         │
│                                                                                                              │
│ Transaction Kind: Programmable                                                                               │
│ ╭──────────────────────────────────────────────────────────────────────────────────────────────────────────╮ │
│ │ Input Objects                                                                                            │ │
│ ├──────────────────────────────────────────────────────────────────────────────────────────────────────────┤ │
│ │ 0   Pure Arg: Type: address, Value: "0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1" │ │
│ ╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯ │
│ ╭─────────────────────────────────────────────────────────────────────────╮                                  │
│ │ Commands                                                                │                                  │
│ ├─────────────────────────────────────────────────────────────────────────┤                                  │
│ │ 0  Publish:                                                             │                                  │
│ │  ┌                                                                      │                                  │
│ │  │ Dependencies:                                                        │                                  │
│ │  │   0x0000000000000000000000000000000000000000000000000000000000000001 │                                  │
│ │  │   0x0000000000000000000000000000000000000000000000000000000000000002 │                                  │
│ │  └                                                                      │                                  │
│ │                                                                         │                                  │
│ │ 1  TransferObjects:                                                     │                                  │
│ │  ┌                                                                      │                                  │
│ │  │ Arguments:                                                           │                                  │
│ │  │   Result 0                                                           │                                  │
│ │  │ Address: Input  0                                                    │                                  │
│ │  └                                                                      │                                  │
│ ╰─────────────────────────────────────────────────────────────────────────╯                                  │
│                                                                                                              │
│ Signatures:                                                                                                  │
│    gebjSbVwZwTkizfYg2XIuzdx+d66VxFz8EmVaisVFiV3GkDay6L+hQG3n2CQ1hrWphP6ZLc7bd1WRq4ss+hQAQ==                  │
│                                                                                                              │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### 交易影响 (Transaction Effects)

交易影响部分包含了交易的状态、交易对网络状态所做的更改以及交易涉及的对象。


```plaintext
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Effects                                                                               │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Digest: GpcDV6JjjGQMRwHpEz582qsd5MpCYgSwrDAq1JXcpFjW                                              │
│ Status: Success                                                                                   │
│ Executed Epoch: 411                                                                               │
│                                                                                                   │
│ Created Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0x160f7856e13b27e5a025112f361370f4efc2c2659cb0023f1e99a8a84d1652f3                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 8                                                                                     │
│  │ Digest: 8y6bhwvQrGJHDckUZmj2HDAjfkyVqHohhvY1Fvzyj7ec                                           │
│  └──                                                                                              │
│  ┌──                                                                                              │
│  │ ID: 0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe                         │
│  │ Owner: Immutable                                                                               │
│  │ Version: 1                                                                                     │
│  │ Digest: Ein91NF2hc3qC4XYoMUFMfin9U23xQmDAdEMSHLae7MK                                           │
│  └──                                                                                              │
│ Mutated Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0x4ea1303e4f5e2f65fc3709bc0fb70a3035fdd2d53dbcff33e026a50a742ce0de                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 8                                                                                     │
│  │ Digest: 7ydahjaM47Gyb33PB4qnW2ZAGqZvDuWScV6sWPiv7LTc                                           │
│  └──                                                                                              │
│ Gas Object:                                                                                       │
│  ┌──                                                                                              │
│  │ ID: 0x4ea1303e4f5e2f65fc3709bc0fb70a3035fdd2d53dbcff33e026a50a742ce0de                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 8                                                                                     │
│  │ Digest: 7ydahjaM47Gyb33PB4qnW2ZAGqZvDuWScV6sWPiv7LTc                                           │
│  └──                                                                                              │
│ Gas Cost Summary:                                                                                 │
│    Storage Cost: 10404400 MIST                                                                    │
│    Computation Cost: 1000000 MIST                                                                 │
│    Storage Rebate: 978120 MIST                                                                    │
│    Non-refundable Storage Fee: 9880 MIST                                                          │
│                                                                                                   │
│ Transaction Dependencies:                                                                         │
│    7Ukrc5GqdFqTA41wvWgreCdHn2vRLfgQ3YMFkdks72Vk                                                   │
│    7d4amuHGhjtYKujEs9YkJARzNEn4mRbWWv3fn4cdKdyh                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### 事件 (Events)

如果有任何 _事件_ 被触发，你将会在这个部分看到它们。由于我们的包没有使用事件，所以这个部分为空。

```plaintext
╭─────────────────────────────╮
│ No transaction block events │
╰─────────────────────────────╯
```

### 对象变更 (Object Changes)

这部分记录了交易所作出的对象变更。在我们的例子中，我们创建了一个新的 `UpgradeCap` 对象，这是一个特殊对象，允许发送者在未来升级包。我们还改变了 Gas 对象，并且发布了一个新的包。在 Sui 中，包也是对象之一。

```plaintext
╭──────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                   │
├──────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                 │
│  ┌──                                                                                             │
│  │ ObjectID: 0x160f7856e13b27e5a025112f361370f4efc2c2659cb0023f1e99a8a84d1652f3                  │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                    │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 ) │
│  │ ObjectType: 0x2::package::UpgradeCap                                                          │
│  │ Version: 8                                                                                    │
│  │ Digest: 8y6bhwvQrGJHDckUZmj2HDAjfkyVqHohhvY1Fvzyj7ec                                          │
│  └──                                                                                             │
│ Mutated Objects:                                                                                 │
│  ┌──                                                                                             │
│  │ ObjectID: 0x4ea1303e4f5e2f65fc3709bc0fb70a3035fdd2d53dbcff33e026a50a742ce0de                  │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                    │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 ) │
│  │ ObjectType: 0x2::coin::Coin<0x2::sui::SUI>                                                    │
│  │ Version: 8                                                                                    │
│  │ Digest: 7ydahjaM47Gyb33PB4qnW2ZAGqZvDuWScV6sWPiv7LTc                                          │
│  └──                                                                                             │
│ Published Objects:                                                                               │
│  ┌──                                                                                             │
│  │ PackageID: 0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe                 │
│  │ Version: 1                                                                                    │
│  │ Digest: Ein91NF2hc3qC4XYoMUFMfin9U23xQmDAdEMSHLae7MK                                          │
│  │ Modules: todo_list                                                                            │
│  └──                                                                                             │
╰──────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### 余额变更 (Balance Changes)

这一部分记录了对 SUI 代币的变动情况。在我们的案例中，我们花费了约 0.015 SUI，相当于 10,500,000 MIST。你可以在输出的 _amount_ 字段中看到这个数值。

```plaintext
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Balance Changes                                                                                   │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│  ┌──                                                                                              │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ CoinType: 0x2::sui::SUI                                                                        │
│  │ Amount: -10426280                                                                              │
│  └──                                                                                              │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
```

### 可选输出

只需在发布时加上 `--json` 标志即可将输出格式设置为 JSON 格式。这对于想要以编程方式解析输出或稍后使用它的人来说非常有用。

```bash
$ sui client publish --gas-budget 100000000 --json
```

### 使用结果

当包成功发布到链上之后，我们可以开始与之进行交互。为了做到这一点，我们需要找到包的地址（对象ID）。这个地址可以在 `Object Changes` 输出的 `Published Objects` 部分找到。每个包的地址都是唯一的，因此您需要从输出中复制它。

在这个示例中，地址是：

```
0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe
```

现在我们有了地址，接下来我们将展示如何通过发送交易与该包进行交互。

### 发送交易

为了演示与`todo_list`包的交互，我们将发送一个交易来创建一个新的列表并向其中添加一个项目。交易通过`sui client ptb`命令发送，它允许充分利用[交易块](./../concepts/what-is-a-transaction.md)。这个命令可能看起来很庞大和复杂，但我们将逐步解释。

### 准备变量

在构建命令之前，让我们存储将在交易中使用的值。将`0x4....`替换为您已发布的包的地址。`MY_ADDRESS`变量将自动从CLI输出中设置为您的地址。

```bash
$ export PACKAGE_ID=0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe
$ export MY_ADDRESS=$(sui client active-address)
```

### 构建CLI中的交易

现在来构建一个实际的交易。该交易将由两部分组成：我们将调用`todo_list`包中的`new`函数来创建一个新的列表，然后将列表对象转移到我们的账户。交易将如下所示：

```bash
$ sui client ptb \
--gas-budget 100000000 \
--assign sender @$MY_ADDRESS \
--move-call $PACKAGE_ID::todo_list::new \
--assign list \
--transfer-objects "[list]" sender
```
在这个命令中，我们使用`ptb`子命令来构建一个交易。随后的参数定义了交易将执行的实际命令和操作。我们首先进行的两个调用是实用函数调用，用于设置发送者地址为命令输入，并为交易设置gas预算。

```bash
# sets the gas budget for the transaction
--gas-budget 100000000 \n
# registers a variable "sender=@..."
--assign sender @$MY_ADDRESS \n
```
接下来，我们执行对包中函数的实际调用。我们使用`--move-call`，紧接着是包ID、模块名和函数名。在这种情况下，我们调用的是`todo_list`包中的`new`函数。

```bash
# calls the "new" function in the "todo_list" package under the $PACKAGE_ID address
--move-call $PACKAGE_ID::todo_list::new
```

我们定义的函数实际上返回一个值，我们需要将其存储起来。我们使用`--assign`命令为返回的值赋予一个名称。在这种情况下，我们将其命名为`list`。然后，我们将该对象转移给我们的账户。

```bash
--move-call $PACKAGE_ID::todo_list::new \
# assigns the result of the "new" function to the "list" variable (from the previous step)
--assign list \
# transfers the object to the sender
--transfer-objects "[list]" sender
```
一旦命令构建完成，您可以在终端中运行它。如果一切正确，您应该看到类似于前面章节中的输出。输出将包含交易摘要、交易数据和交易效果。

<details>
<summary><a style="cursor:pointer">剧透：完整的交易输出</a></summary>

```bash
Transaction Digest: BJwYEnuuMzU4Y8cTwMoJbbQA6cLwPmwxvsRpSmvThoK8
╭──────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Data                                                                                             │
├──────────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                                   │
│ Gas Owner: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                                │
│ Gas Budget: 100000000 MIST                                                                                  │
│ Gas Price: 1000 MIST                                                                                         │
│ Gas Payment:                                                                                                 │
│  ┌──                                                                                                         │
│  │ ID: 0xe5ddeb874a8d7ead328e9f2dd2ad8d25383ab40781a5f1aefa75600973b02bc4                                    │
│  │ Version: 22                                                                                               │
│  │ Digest: DiBrBMshDiD9cThpaEgpcYSF76uV4hCoE1qRyQ3rnYCB                                                      │
│  └──                                                                                                         │
│                                                                                                              │
│ Transaction Kind: Programmable                                                                               │
│ ╭──────────────────────────────────────────────────────────────────────────────────────────────────────────╮ │
│ │ Input Objects                                                                                            │ │
│ ├──────────────────────────────────────────────────────────────────────────────────────────────────────────┤ │
│ │ 0   Pure Arg: Type: address, Value: "0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1" │ │
│ ╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯ │
│ ╭──────────────────────────────────────────────────────────────────────────────────╮                         │
│ │ Commands                                                                         │                         │
│ ├──────────────────────────────────────────────────────────────────────────────────┤                         │
│ │ 0  MoveCall:                                                                     │                         │
│ │  ┌                                                                               │                         │
│ │  │ Function:  new                                                                │                         │
│ │  │ Module:    todo_list                                                          │                         │
│ │  │ Package:   0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe │                         │
│ │  └                                                                               │                         │
│ │                                                                                  │                         │
│ │ 1  TransferObjects:                                                              │                         │
│ │  ┌                                                                               │                         │
│ │  │ Arguments:                                                                    │                         │
│ │  │   Result 0                                                                    │                         │
│ │  │ Address: Input  0                                                             │                         │
│ │  └                                                                               │                         │
│ ╰──────────────────────────────────────────────────────────────────────────────────╯                         │
│                                                                                                              │
│ Signatures:                                                                                                  │
│    C5Lie4dtP5d3OkKzFBa+xM0BiNoB/A4ItthDCRTRBUrEE+jXeNs7mP4AuGwi3nzfTskh29+R1j1Kba4Wdy3QDA==                  │
│                                                                                                              │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Transaction Effects                                                                               │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Digest: BJwYEnuuMzU4Y8cTwMoJbbQA6cLwPmwxvsRpSmvThoK8                                              │
│ Status: Success                                                                                   │
│ Executed Epoch: 1213                                                                              │
│                                                                                                   │
│ Created Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0x74973c4ea2e78dc409f60481e23761cee68a48156df93a93fbcceb77d1cacdf6                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 23                                                                                    │
│  │ Digest: DuHTozDHMsuA7cFnWRQ1Gb8FQghAEBaj3inasJxqYq1c                                           │
│  └──                                                                                              │
│ Mutated Objects:                                                                                  │
│  ┌──                                                                                              │
│  │ ID: 0xe5ddeb874a8d7ead328e9f2dd2ad8d25383ab40781a5f1aefa75600973b02bc4                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 23                                                                                    │
│  │ Digest: 82fwKarGuDhtomr5oS6ZGNvZNw9QVXLSbPdQu6jQgNV7                                           │
│  └──                                                                                              │
│ Gas Object:                                                                                       │
│  ┌──                                                                                              │
│  │ ID: 0xe5ddeb874a8d7ead328e9f2dd2ad8d25383ab40781a5f1aefa75600973b02bc4                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ Version: 23                                                                                    │
│  │ Digest: 82fwKarGuDhtomr5oS6ZGNvZNw9QVXLSbPdQu6jQgNV7                                           │
│  └──                                                                                              │
│ Gas Cost Summary:                                                                                 │
│    Storage Cost: 2318000 MIST                                                                     │
│    Computation Cost: 1000000 MIST                                                                 │
│    Storage Rebate: 978120 MIST                                                                    │
│    Non-refundable Storage Fee: 9880 MIST                                                          │
│                                                                                                   │
│ Transaction Dependencies:                                                                         │
│    FSz2fYXmKqTf77mFXNq5JK7cKY8agWja7V5yDKEgL8c3                                                   │
│    GgMZKTt482DYApbAZkPDtdssGHZLbxgjm2uMXhzJax8Q                                                   │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─────────────────────────────╮
│ No transaction block events │
╰─────────────────────────────╯

╭───────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                        │
├───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                      │
│  ┌──                                                                                                  │
│  │ ObjectID: 0x74973c4ea2e78dc409f60481e23761cee68a48156df93a93fbcceb77d1cacdf6                       │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )      │
│  │ ObjectType: 0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList  │
│  │ Version: 23                                                                                        │
│  │ Digest: DuHTozDHMsuA7cFnWRQ1Gb8FQghAEBaj3inasJxqYq1c                                               │
│  └──                                                                                                  │
│ Mutated Objects:                                                                                      │
│  ┌──                                                                                                  │
│  │ ObjectID: 0xe5ddeb874a8d7ead328e9f2dd2ad8d25383ab40781a5f1aefa75600973b02bc4                       │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )      │
│  │ ObjectType: 0x2::coin::Coin<0x2::sui::SUI>                                                         │
│  │ Version: 23                                                                                        │
│  │ Digest: 82fwKarGuDhtomr5oS6ZGNvZNw9QVXLSbPdQu6jQgNV7                                               │
│  └──                                                                                                  │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭───────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Balance Changes                                                                                   │
├───────────────────────────────────────────────────────────────────────────────────────────────────┤
│  ┌──                                                                                              │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )  │
│  │ CoinType: 0x2::sui::SUI                                                                        │
│  │ Amount: -2339880                                                                               │
│  └──                                                                                              │
╰───────────────────────────────────────────────────────────────────────────────────────────────────╯
```

</details>

我们要关注的部分是"Object Changes"（对象变化）部分，更具体地说是其中的"Created Objects"（创建的对象）部分。它包含了您创建的 `TodoList` 的对象 ID、类型和版本信息。我们将使用这个对象 ID 来与对应列表进行交互。

```bash
╭───────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Object Changes                                                                                        │
├───────────────────────────────────────────────────────────────────────────────────────────────────────┤
│ Created Objects:                                                                                      │
│  ┌──                                                                                                  │
│  │ ObjectID: 0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553                       │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )      │
│  │ ObjectType: 0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList  │
│  │ Version: 22                                                                                        │
│  │ Digest: HyWdUpjuhjLY38dLpg6KPHQ3bt4BqQAbdF5gB8HQdEqG                                               │
│  └──                                                                                                  │
│ Mutated Objects:                                                                                      │
│  ┌──                                                                                                  │
│  │ ObjectID: 0xe5ddeb874a8d7ead328e9f2dd2ad8d25383ab40781a5f1aefa75600973b02bc4                       │
│  │ Sender: 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1                         │
│  │ Owner: Account Address ( 0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1 )      │
│  │ ObjectType: 0x2::coin::Coin<0x2::sui::SUI>                                                         │
│  │ Version: 22                                                                                        │
│  │ Digest: DiBrBMshDiD9cThpaEgpcYSF76uV4hCoE1qRyQ3rnYCB                                               │
│  └──                                                                                                  │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────╯
```
在这个示例中，对象 ID 是`0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553`。而拥有者应该是您的账户地址。我们通过在交易的最后一个命令中将对象转移给发送者来实现这一点。

另一种测试您是否成功创建了列表的方法是检查账户对象。

```bash
$ sui client objects
```

它应该有一个类似于以下内容的对象：

```plaintext
╭  ...                                                                                  ╮
│ ╭────────────┬──────────────────────────────────────────────────────────────────────╮ │
│ │ objectId   │  0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553  │ │
│ │ version    │  22                                                                  │ │
│ │ digest     │  /DUEiCLkaNSgzpZSq2vSV0auQQEQhyH9occq9grMBZM=                        │ │
│ │ objectType │  0x468d..29fe::todo_list::TodoList                                   │ │
│ ╰────────────┴──────────────────────────────────────────────────────────────────────╯ │
|  ...                                                                                  |
```

### 将对象传递给函数

我们在上一步中创建的TodoList是一个可以作为其所有者进行交互的对象。您可以在该对象上调用`todo_list`模块中定义的函数。为了演示这一点，我们将向列表中添加一个项目。首先，我们只添加一个项目，然后在第二个交易中添加3个项目并删除另一个项目。

请再次检查您是否已经设置了[前一步骤中的变量](#准备变量)，然后为列表对象添加一个变量。

```bash
$ export LIST_ID=0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553
```

现在我们可以构建一个交易，向列表中添加一个项目。命令将如下所示：

```bash
$ sui client ptb \
--gas-budget 100000000 \
--move-call $PACKAGE_ID::todo_list::add @$LIST_ID "'Finish the Hello, Sui chapter'"
```

在这个命令中，我们调用了 `todo_list` 包中的 `add` 函数。该函数接受两个参数：列表对象和要添加的项目。项目是一个字符串，所以我们需要用单引号将其包裹起来。该命令将项目添加到列表中。

如果一切正确，您应该看到类似于前面章节中的输出。现在，您可以检查列表对象，看看项目是否已经被添加进去了。

```bash
$ sui client object $LIST_ID
```

输出应该包含您添加的项目。

```plaintext
╭───────────────┬───────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ objectId      │  0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553                                               │
│ version       │  24                                                                                                               │
│ digest        │  FGcXH8MGpMs5BdTnC62CQ3VLAwwexYg2id5DKU7Jr9aQ                                                                     │
│ objType       │  0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList                          │
│ owner         │ ╭──────────────┬──────────────────────────────────────────────────────────────────────╮                           │
│               │ │ AddressOwner │  0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1  │                           │
│               │ ╰──────────────┴──────────────────────────────────────────────────────────────────────╯                           │
│ prevTx        │  EJVK6FEHtfTdCuGkNsU1HcrmUBEN6H6jshfcptnw8Yt1                                                                     │
│ storageRebate │  1558000                                                                                                          │
│ content       │ ╭───────────────────┬───────────────────────────────────────────────────────────────────────────────────────────╮ │
│               │ │ dataType          │  moveObject                                                                               │ │
│               │ │ type              │  0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList  │ │
│               │ │ hasPublicTransfer │  true                                                                                     │ │
│               │ │ fields            │ ╭───────┬───────────────────────────────────────────────────────────────────────────────╮ │ │
│               │ │                   │ │ id    │ ╭────┬──────────────────────────────────────────────────────────────────────╮ │ │ │
│               │ │                   │ │       │ │ id │  0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553  │ │ │ │
│               │ │                   │ │       │ ╰────┴──────────────────────────────────────────────────────────────────────╯ │ │ │
│               │ │                   │ │ items │ ╭─────────────────────────────────╮                                           │ │ │
│               │ │                   │ │       │ │  finish the Hello, Sui chapter  │                                           │ │ │
│               │ │                   │ │       │ ╰─────────────────────────────────╯                                           │ │ │
│               │ │                   │ ╰───────┴───────────────────────────────────────────────────────────────────────────────╯ │ │
│               │ ╰───────────────────┴───────────────────────────────────────────────────────────────────────────────────────────╯ │
╰───────────────┴───────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

可以通过在命令中添加`--json`标志来获取对象的JSON表示形式。

```bash
$ sui client object $LIST_ID --json
```

```json
{
  "objectId": "0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553",
  "version": "24",
  "digest": "FGcXH8MGpMs5BdTnC62CQ3VLAwwexYg2id5DKU7Jr9aQ",
  "type": "0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList",
  "owner": {
    "AddressOwner": "0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1"
  },
  "previousTransaction": "EJVK6FEHtfTdCuGkNsU1HcrmUBEN6H6jshfcptnw8Yt1",
  "storageRebate": "1558000",
  "content": {
    "dataType": "moveObject",
    "type": "0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList",
    "hasPublicTransfer": true,
    "fields": {
      "id": {
        "id": "0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553"
      },
      "items": ["Finish the Hello, Sui chapter"]
    }
  }
}
```

### 链式命令

链接多个命令可以在单个交易中执行。这展示了交易块的威力！使用相同的列表对象，我们将添加三个项目并删除一个项目。命令将如下所示：

```bash
$ sui client ptb \
--gas-budget 100000000 \
--move-call $PACKAGE_ID::todo_list::add @$LIST_ID "'Finish Concepts chapter'" \
--move-call $PACKAGE_ID::todo_list::add @$LIST_ID "'Read the Move Basics chapter'" \
--move-call $PACKAGE_ID::todo_list::add @$LIST_ID "'Learn about Object Model'" \
--move-call $PACKAGE_ID::todo_list::remove @$LIST_ID 0
```

如果之前的命令都成功执行了，这个命令也应该不会有问题。你可以检查列表对象，看看项目是否已被添加和删除。JSON 格式会更易读一些！

```bash
sui client object $LIST_ID --json
```

```json
{
  "objectId": "0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553",
  "version": "25",
  "digest": "EDTXDsteqPGAGu4zFAj5bbQGTkucWk4hhuUquk39enGA",
  "type": "0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList",
  "owner": {
    "AddressOwner": "0x091ef55506ad814920adcef32045f9078f2f6e9a72f4cf253a1e6274157380a1"
  },
  "previousTransaction": "7SXLGBSh31jv8G7okQ9mEgnw5MnTfvzzHEHpWf3Sa9gY",
  "storageRebate": "1922800",
  "content": {
    "dataType": "moveObject",
    "type": "0x468daa33dfcb3e17162bbc8928f6ec73744bb08d838d1b6eb94eac99269b29fe::todo_list::TodoList",
    "hasPublicTransfer": true,
    "fields": {
      "id": {
        "id": "0x20e0bede16de8a728ab25e228816b9059b45ebea49c8ad384e044580b2d3e553"
      },
      "items": [
        "Finish Concepts chapter",
        "Read the Move Basics chapter",
        "Learn about Object Model"
      ]
    }
  }
}
```

命令不必在同一个包中，也不必操作同一个对象。在单个交易块中，你可以与多个包和对象进行交互。这是一个强大的功能，可以让你在链上构建复杂的交互！

## 总结

在本指南中，我们展示了如何在 Move 区块链上发布包并使用 Sui CLI 与其交互。我们演示了如何创建一个新的列表对象、添加项目以及删除项目。我们还展示了如何在单个交易块中链接多个命令。本指南应该能为你在 Sui 区块链上构建自己的应用程序提供一个良好的起点！
