
# ERC 

[ERC(Ethereum Request for Comments)](https://eips.ethereum.org/erc) 标准定义了一套 Ethereum 区  Token 的标准。块链上用于创建和管理的规则。

> 所有 ERC 标准都需要通过 EIP(Ethereum Improvement Proposals，以太坊改进方案)进行修订，最后社区讨论共同决定


常见的 ERC 标准：

- [ERC-20](https://eips.ethereum.org/EIPS/eip-20)，用于在 Ethereum 区块链上创建代币的技术标准
- ERC-621，是 ERC-20 的扩展，新增了“允许 Token 发行者对流通中的 Token 总量增加和减少”的功能，这对于管理代币经济和确保价格稳定很有用
- [ERC-165](https://eips.ethereum.org/EIPS/eip-165)，标准接口检测。创建了一个标准方法来发布和检测智能合约实现的接口
- [ERC-721](https://eips.ethereum.org/EIPS/eip-721)，定义了 NFT(Non-Fungible Token) 非同质化代币，不同于 ERC-20 Token，每个 ERC-721 Token 都是唯一且不可互换的，每个 NFT 都有一个 `uint256` 的 ID
  - 必须搭配 ERC-165 来使用
- ERC-721x 是 ERC-721 的扩展，增加了对 Token 的批量替代和传输功能
- [ERC-777](https://eips.ethereum.org/EIPS/eip-777)，相比于 ERC-20，它定义了与 Token 交互的高级功能（授权、撤销、转移、检查等）
  - 使用了 ERC-1820，在智能合约中注册元数据，以便和之前版本的 Token 实现向后兼容
- [ERC-884](https://eips.ethereum.org/EIPS/eip-884)，通过添加对资产部分所有权的支持来扩展 ERC-20 和 ERC-721 标准的功能
  - 主要优点之一是它允许在区块链上表示现实世界资产（例如财产或商品）的部分所有权
- [ERC-1155](https://eips.ethereum.org/EIPS/eip-1155)，用于管理多种 Token 类型（传统 Token、NFT、半同质化代币等）的合约接口
  - 传统上，创建管理多种类型代币的 dApp 需要为每种类型制定单独的智能合约，这可能很麻烦且效率低下，且会在以太坊区块链上放置大量冗余字节码，且随着 gameFi 的兴起，游戏开发者可能会一次性创建数以千计的 Token。借助 ERC-1155，开发人员可以创建一个管理可替代和不可替代代币的智能合约，从而降低 dApp 的整体复杂性。
  - ERC-1155 的另一个优点是它允许创建“半同质”代币。这些代币具有一些可替代和不可替代的属性。例如，一个游戏物品可能具有一组该物品独有的属性（使其不可替代），但也具有一组在多个物品之间共享的属性（使其可替代）。这使得管理游戏内物品变得更加容易，并为游戏开发者提供了更大的灵活性。
  - ERC-1155 还支持“批量转账”，允许在一次交易中转账多个代币。这可以在转移大量代币时降低 gas 成本并提高整体效率。

Reference：

[All ERC Token Standards Explained](https://web3.career/learn-web3/erc-token-standards)

