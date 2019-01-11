# EOSForce Introduction

<!-- vim-markdown-toc GFM -->

* [Preface](#Preface)
* [Model](#Model)
    * [User Assets](#User Assets)
    * [Block Rewards](#Block Rewards)
    * [Transaction Fees](#Transaction Fees)
* [Governance](#Governance)
    * [Voting Dividends](#Voting Dividends)
    * [One Vote One Vote](#One Vote One Vote)
    * [23 Block Producers](#23 Block Producers)
    * [Emergency State](#Emergency State)
* [Launch](#Launch)
* [Conclusions](#Conclusions)
* [Thanks](#Thanks)

<!-- vim-markdown-toc -->

[English Version](README_en.md)

## Preface

EOS Force is EOSIO-based blockchain software (https://github.com/EOSIO/eos) and will evolve from it。

During deep research of EOSIO software，we found many factors that could result in instability，such as WAVM-based smart contract's security developped by C++，"super node alliance" resulting from one vote thirty votes etc. If we lauch the mainnet、elect the BPs、vote and deploy smart contracts according to official's guide [EOSIO](https://github.com/EOSIO) ，then users' assets could be difficult to be guaranteed。

In line with the principle of user asset security as the first element，we evolve from EOSIO [EOSIO/eos](https://github.com/EOSIO/eos) and finally proposed EOS Force solutions。

By adjusting block production interval，levying transaction fees，encouraging BPS to distribute dividends，Releasing self-deploy smart contracts functions gradually，EOS Force was devoted to increase the blockchain's stability and security。

## Model

### User Assets

EOS Force support mapping of EOS ERC20 tokens on ETH blockchain，user assets on EOS Force blockchain has following attributes：

- Available balance：could be transfered、voted。
- Voting amount：amount voting for different BPs，处于锁定状态，减少投票后变为赎回金额。
- 赎回金额：撤销的投票金额，有 3 天冻结期，3 天后可以提取成可用余额。
- 待领分红：用户根据对节点选举的币量和时间贡献，占有节点奖励池的一部分，提取后变为可用余额。

### Block Rewards

EOSIO 默认 0.5s 的出块速度在全球性的分布式网络中尚未得到有效验证，网络延迟很可能会造成区块链分叉和停止。因此，EOS Force 在链的启动阶段将出块时间设为 3 秒，每个节点每次只出一个块，每个块奖励为 9 个 EOS。待链运行稳定后，EOS Force 将会恢复 0.5s 的出块时间，在稳定的基础上进一步提升链的性能。

### 交易手续费

EOSIO 需要用户抵押币来获取资源，从而竞争性地使用区块链，继而达到 “免交易手续费” 的目的。“免交易手续费” 实际是自欺欺人，超级节点可获得 1% 的年化奖励，这实质上将交易手续费转嫁为了用户必须承担的 1% 年化通胀。

为了链的安全性，防止被 DDOS 攻击，EOS Force 恢复了交易手续费，以交易执行的种类计费，用户无需指定手续费金额，系统将会自动从交易发起方的余额中扣除，如果余额不足，交易失败。

通过不同合约执行不同操作会收取相应手续费。通过 eosio 进行 EOS 的转账、解除冻结，以及 eosio.token 进行 token 的发行与转账，这些交易的手续费都是 0.01 EOS。投票的手续费为 0.05 EOS，领取分红的手续费为 0.03 EOS。通过 eosio 设置紧急状态以及 eosio.token 创建新的 token 手续费为 10 EOS。通过 eosio 注册 BP 的手续费为 100 EOS。

## 治理

### 投票分红

如果不给投票用户分红，普通用户的投票意愿就会降低，这会导致全链币的投票比例降低，那么几个大户联合就可能操纵投票影响选举，从而进行分叉攻击。所以，我们鼓励超级节点给投票的用户进行分红，充分活跃普通用户的投票参与度。

EOS Force 每年大约有 9000 万 EOS 奖励，超级节点可以自行设置自己的佣金比例，比如 1%。那么节点当选并出块后，可以拿走每个块奖励的 1%，剩余 99% 会进入每个节点的奖励池。节点根据每个用户的投票金额和时间得出用户“票龄”，再根据节点所有用户的“总票龄”，计算出每个用户在奖励池中的分红占比，给节点投票的用户随时可以从奖励池中提取分红。

如果 EOS Force 全网只有 3 亿的 EOS 参与投票，那么所有这些投票用户将平分 9000 万 EOS 的奖励，年化利率约为 0.9亿/3亿，也就是 30%。用户的年化利率随着投票参与率的升高而降低。随着币总量的上升，每年的奖励比例也会逐年下降。

为了减少自动分发消耗大量运算资源，EOS Force 的投票分红需要用户手动领取，领取快慢并不影响分红数量，所提取分红会立即变成可用余额。

用户每次提取分红后，在节点中的“票龄”会归零重新累计。

### 一票一投

EOS Force 实行一票一投的用户投票机制，1 个 EOS 只能投给某一个节点，但是一个用户可以给多个节点分别投不同数量的币。

假设一个用户有 1000 个 EOS，节点 A 的佣金比例是 1%，用户投给 A 300 个 EOS，节点 B 的佣金比例是 1.5%，用户投给 B 100 个 EOS，那么该用户的可用余额还剩 600 个 EOS，用户最终可以从这两个节点分别获得相应的投票分红。

EOS Force 支持用户调整投票数量，即增加或减少投票。如果增加投票，则自动进行一次分红领取，并扣除可用余额。如果减少投票，也会自动进行一次分红领取，同时减少的币量会有 3 天的冻结时间，3 天后，用户需要手动进行“解除冻结”操作，才能把投票金额变为可用余额。

### 23 个超级节点

在 BFT 算法中，节点数天然不适合是 3 的整数倍。如果是 21 个节点，且恰好形成了 14 票同意，7 票反对的局面，则既无法达成大于 2/3 的通过，也无法达成大于 1/3 的否决，治理陷入僵局。如果是 23 个节点，不是 3 的整数倍，那么最终会形成 15 票同意，8 票反对的否决决定，或者 16 票同意，7 票反对的通过决定，不会形成僵局。

### 紧急状态

EOS 链还不能完全确认稳定性，如果链出现没有预料到的 BUG，那么需要有可以设置紧急状态的功能，使链进入超级节点治理状态。一旦进入紧急状态，立即停止转账、投票、分红等可能影响用户资产安全的操作，只允许节点治理相关的操作。只有注册节点可以开启和关闭紧急状态，如果在职 23 个节点中有 16 个节点同意开启，则紧急状态启动。问题处理后，超级节点可以选择关闭紧急状态，当关闭紧急状态的节点数大于 8 时，恢复链的全部功能。

## 启动

- 启动阶段追求链的稳定性。在此阶段，只有用户转账、投票、分红的系统合约，用户不能自主部署新合约。

- 基础功能稳定后，开放合约部署功能，开发者可以开发 DAPP。

- 网络稳定后，各超级节点也完成了前期训练，恢复 0.5s 的出块速度。

EOS Force 创世块中默认会有 23 个引导节点，链启动后超级节点即可注册参与出块。

## 结束语

安全稳定是区块链的首要追求，EOS Force 将与社区一起为实现这一目标而不断努力！

## 致谢

- [EOSIO](https://github.com/EOSIO/eos)