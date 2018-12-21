# Eosforce资源模型介绍

## 1. Eosforce中的资源

在Eosforce中每一个执行的交易都需要Eosforce网络中的节点去运行，节点运行交易得到结果是需要由实际的物理机器去计算，
这类计算所需的资源被归为三类：CPU，NET和RAM，其中CPU以节点执行交易所用的时间来结算，NET是以交易消息大小来结算，
而RAM，意思是内存，是以交易产生的数据的大小来结算，这些数据需要节点存贮在节点的内存中，以供其他合约读写。

因为节点的物理机器的计算能力和存储能力都是有限的，所以整个Eosforce网络中用户能使用的资源也是有限的，
为了使得所有用户都能使用到资源来进行交易，防止某些用户无度的滥用计算资源而导致其他用户无法交易，
需要建立一系列资源的分配规则，这就是Eosforce的资源模型。

针对现在EMLG EOS主网运行中产生的一系列问题，Eosforce对EOS进行了修改，建立了一套新的资源模型，
在保证资源分配公平合理的情况下，尽量减少对于资源的滥用，让真正需要资源的用户可以使用Eosforce网络提供的资源。

下面就是对这一资源模型的介绍。

## 2. 基于手续费分配CPU和NET资源

在Eosforce中，我们需要用户为其所触发的每一个action支付手续费，这一方式类似与以太坊。
每一笔手续费会为其action提供一个CPU和NET的资源使用上限，
对于系统原生的action如转账、创建用户、更新权限等action，则是采用固定手续费和限制的方式，便于用户使用，
对于用户定义的action，其中的换算比例是由BP通过多签来设置的，在默认情况下，每支付0.01 EOSC，该action会被赋予 200 us cpu使用限制和 500 byte net使用限制，为了便于用户使用，开发者需要为自己提交的合约中action设置手续费额度，这样用户则不需自行设置手续费额度，同时也激励开发者优化合约资源使用效率，为用户提供更好的体验。

如下面的过程：

```bash
cleost transfer eosforce test "5000.0000 EOS" "fo test"
executed transaction: ed62211eda722230472d416a8e3c92ab3f950fe951bcd357f9fb217792dac936  152 bytes  213 us
#         eosio <= eosio::onfee                 {"actor":"eosforce","fee":"0.0100 EOS","bpname":"biosbpb"}
#         eosio <= eosio::transfer              {"from":"eosforce","to":"test","quantity":"5000.0000 EOS","memo":"fo test"}
#      eosforce <= eosio::transfer              {"from":"eosforce","to":"test","quantity":"5000.0000 EOS","memo":"fo test"}
#          test <= eosio::transfer              {"from":"eosforce","to":"test","quantity":"5000.0000 EOS","memo":"fo test"}
```

我们用`eosforce`账户给`test`账户转账了5000 EOSC，这个交易中只运行了 `eosio::transfer` action，
所以对于这个action执行了对应的`eosio::onfee`action来向`eosforce`账户收取了 0.01 EOSC作为交易的手续费。

注意 `eosio::transfer` action会触发`eosforce <= eosio::transfer`和`test <= eosio::transfer`两个通知，但是这里两个账户都没用相应通知，所以这里只执行了一个action。

Eosforce会为transaction中的所有action分别计算手续费，这里的action也包括inline action，

如下面，这里`testc`是合约账户，当接受到转账时，会向另外两个账户`testa`和`testb`转账，这里构建一个transaction，向`testc`账户转账：

```bash
cleost transfer eosforce testc "5.0000 EOS" "fo test"  
executed transaction: 6339c73f167602607c8ee1a89a73e7f0171f375b39b361eb47047f0e4a48c69d  152 bytes  646 us
#         eosio <= eosio::onfee                 {"actor":"eosforce","fee":"0.0100 EOS","bpname":"biosbpm"}
#         eosio <= eosio::transfer              {"from":"eosforce","to":"testc","quantity":"5.0000 EOS","memo":"fo test"}
#      eosforce <= eosio::transfer              {"from":"eosforce","to":"testc","quantity":"5.0000 EOS","memo":"fo test"}
#         testc <= eosio::transfer              {"from":"eosforce","to":"testc","quantity":"5.0000 EOS","memo":"fo test"}
>> apply testc eosio transfer
#         eosio <= eosio::onfee                 {"actor":"eosforce","fee":"0.0100 EOS","bpname":"biosbpm"}
#         eosio <= eosio::transfer              {"from":"testc","to":"testa","quantity":"5.0000 EOS","memo":"1;testa"}
#         testc <= eosio::transfer              {"from":"testc","to":"testa","quantity":"5.0000 EOS","memo":"1;testa"}
>> apply testc eosio transfer
#         testa <= eosio::transfer              {"from":"testc","to":"testa","quantity":"5.0000 EOS","memo":"1;testa"}
#         eosio <= eosio::onfee                 {"actor":"eosforce","fee":"0.0100 EOS","bpname":"biosbpm"}
#         eosio <= eosio::transfer              {"from":"testc","to":"testb","quantity":"5.0000 EOS","memo":"1;testa"}
#         testc <= eosio::transfer              {"from":"testc","to":"testb","quantity":"5.0000 EOS","memo":"1;testa"}
>> apply testc eosio transfer
#         testb <= eosio::transfer              {"from":"testc","to":"testb","quantity":"5.0000 EOS","memo":"1;testa"}
warning: transaction executed locally, but may not be confirmed by the network yet         ] 
```

这个过程中，首先是`eosforce`向`testc`转账需要0.01 EOSC手续费，之后`testc`的合约触发了两个inline_action，分别是向`testa`和`testb`转账，这里由收取两次0.01 EOSC手续费，所以整个合约会收取0.03 EOSC手续费。

## 3. 基于租赁分配RAM资源

与CPU和NET不同，RAM采用租赁方式，以账户为单位根据其所缴纳租金来设置账户可以使用的RAM的上限，这里租金按照时间收取，为了方便用户操作，现阶段我们为每个账户设置了8kb的免租金额度，这样对于绝大多数普通用户是不需要关心RAM的，通常，开发者需要为其DApp所使用的RAM支付租金，包括了合约占据的RAM和执行中产生的RAM数据。

目前我们实现了通过投票分红来支付租金的功能，在Eosforce中用户投票可以获得分红，这形成了一个token流，而RAM租金也是按照时间和租赁的多少来计费，通过分红支付租金，一方面方便用户操作，不需用户经常补交租金和缴纳押金，另一方面可以提高投票率，促进链生态健康发展。

这个过程是这样的：

```
用户 --> 投票 --> 每一段时间会有分红 --> 支付租金 --> 获取RAM额度
```

为了简化计算开销，系统会设置每投票多少EOS获得多少RAM的租金费率，这一费率默认是每投票10 EOSC可以获得1kb的RAM使用额度， 即投票10000 EOSC可以获取1MB的RAM使用额度，BP可以通过多签来修改这一费率，使得BP可以在节点成本降低后，降低费率。

### 3.1. 钱包使用流程 TODO

### 3.2. 命令行使用流程

通过投票分红抵扣RAM租金可以通过 `eosio::vote4ram` 和 `eosio::unfreezeram` 来设置投票额和解冻撤回的投票，操作和 `eosio::vote` 和 `eosio::unfreeze`类似：

一般一个用户会有8kb的免费额度，可以通过获取用户信息查看：

```bash
cleos get account testc
created: 2018-05-28T12:00:00.000
permissions: 
     owner     1:    1 EOS65jXw13YBGKGbKiJsFiq2TmnnYsAuaKEomqxZwu2xSQsd15D8q
        active     1:    1 EOS65jXw13YBGKGbKiJsFiq2TmnnYsAuaKEomqxZwu2xSQsd15D8q
memory: 
     quota:         8 KiB    used:      2.66 KiB  

net bandwidth: 
     used:               unlimited
     available:          unlimited
     limit:              unlimited

cpu bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited
```

每个用户注册后会由8kb的使用额度，注意一些固有的用户信息会占用2.66kb。

账户 `testc` 向 `biosbpd` 投 1000 EOSC 来获取分红：

```bash
cleos push action eosio vote4ram '{"voter":"testc","bpname":"biosbpd","stake":"1000.0000 EOS"}' -p testc
```

通过获取用户信息可以查看此时的用户的RAM额度：

```bash
cleost get account testc
created: 2018-05-28T12:00:00.000
permissions: 
     owner     1:    1 EOS65jXw13YBGKGbKiJsFiq2TmnnYsAuaKEomqxZwu2xSQsd15D8q
        active     1:    1 EOS65jXw13YBGKGbKiJsFiq2TmnnYsAuaKEomqxZwu2xSQsd15D8q
memory: 
     quota:       108 KiB    used:      2.66 KiB  

net bandwidth: 
     used:               unlimited
     available:          unlimited
     limit:              unlimited

cpu bandwidth:
     used:               unlimited
     available:          unlimited
     limit:              unlimited
```

这时可以通过`eosio`下的`votes4ram`表来查询一个用户使用分红支付租金的投票情况：

```bash
cleos get table eosio testc votes4ram 
{
  "rows": [{
      "bpname": "biosbpd",
      "staked": "1000.0000 EOS",
      "voteage": 572500,
      "voteage_update_height": 637,
      "unstaking": "1500.0000 EOS",
      "unstake_height": 560
    }
  ],
  "more": false
}
```

账户 `testc` 撤回投 `biosbpd` 的 300 EOSC ， 也就是设置为投 700 EOSC(1000 EOSC - 300 EOSC)：

```bash
cleos push action eosio vote4ram '{"voter":"testc","bpname":"biosbpd","stake":"700.0000 EOS"}' -p testc
```

在超过冷冻时间之后，可以通过`eosio::unfreezeram`来解冻：

```bash
cleos push action eosio unfreezeram '{"voter":"testc","bpname":"biosbpd"}' -p testc
```