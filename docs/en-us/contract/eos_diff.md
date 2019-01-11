# 合约开发

## 合约编译上传流程

```shell
account='你的合约账号'
eosiocpp=./build/tools/eosiocpp #eosiocpp工具路径，可由eosforce项目编译后产生
contract_path='你的合约源码路径'

# 编译
$eosiocpp -g contract.abi $contract_path/contract.hpp
$eosiocpp -o contract.wasm $contract_path/contract.cpp

# 上传
leos wallet unlock --password '钱包密码'
cleos -u https://w1.eosforce.cn set abi $account contract.abi
cleos -u https://w1.eosforce.cn set code $account contract.wasm
```

## 与EMLG EOS的不同点，与合约迁移注意事项

### 1. transfer 转账

transfer转账消息监听，或合约内EOS系统代币transfer转账操作，使用的合约账号code是‘eosio’，而不是‘eosio.token’。 
因为原力的EOSC是在系统合约内，与‘eosio.token’合约是分离的。token代币逻辑不变。

eg：合约内inline action转账

```c++
action(permission_level{_self, N(active)}, N(eosio), N(transfer),std::make_tuple(_self, to, quantity, std::string("")))
        .send();
```

注：进行合约内转账，合约账号的active权限需要授权给eosio.code账号。

```shell
account='你的合约账号'
PK='合约账户公钥'
cleos set account permission $account active '{"threshold": 1,"keys": [{"key":"'$PK'", "weight":1}],"accounts": [{"permission":{"actor":"'$account'","permission":"eosio.code"},"weight":1}]}' owner -p $account'@owner'
```

### 2. 手续费机制。

首先，可以参考 [资源模型介绍文档](https://eosforce.github.io/Documentation/#/zh-cn/res_limit) 。

在EOSForce中每个action执行时需要预先设置手续费，如果一个action没有指定手续费，则会导致如下错误：

```shell
error 2019-01-11T07:23:20.785 thread-0  http_plugin.cpp:580           handle_exception     ] FC Exception encountered while processing chain.push_transaction
debug 2019-01-11T07:23:20.785 thread-0  http_plugin.cpp:581           handle_exception     ] Exception Details: 3050000 action_validate_exception: Action validate exception
action testd hi name not include in feemap or db
    {"acc":"testd","act":"hi"}
    thread-0  txfee_manager.cpp:76 get_required_fee

```

这里是调用部署在`testd`账户中的`hi`action，在请求的节点的日志中会详细打印信息。

在Eosforce中，我们需要用户为其所触发的每一个action支付手续费，这一方式类似与以太坊。 每一笔手续费会为其action提供一个CPU和NET的资源使用上限， 对于系统原生的action如转账、创建用户、更新权限等action，则是采用固定手续费和限制的方式，便于用户使用， 对于用户定义的action，其中的换算比例是由BP通过多签来设置的，在默认情况下，每支付0.01 EOSC，该action会被赋予 200 us cpu使用限制和 500 byte net使用限制，为了便于用户使用，开发者需要为自己提交的合约中action设置手续费额度，这样用户则不需自行设置手续费额度，同时也激励开发者优化合约资源使用效率，为用户提供更好的体验。

对于DApp开发者来说，首先需要评估自己合约action所消耗的资源，之后自行为自己的账户设置手续费：

使用 setfee：

```shell
./cleos set setfee testd hi "0.0500 EOS" 0 0 0
debug 2019-01-11T07:27:14.427 thread-0  main.cpp:2465                 operator()           ] set fee hi, 0.0500 SYS
executed transaction: 247de68c2815eec4fa46b1356bd6bec247a43aa2c22f68c6cc976cf6accc9a4d  136 bytes  250 us
#         eosio <= eosio::setfee                "000000008094b1ca000000000000806bf4010000000000000453595300000000000000000000000000000000"
#   eosio.token <= eosio.token::fee             {"payer":"testd","quantity":"0.1000 EOS"}
warning: transaction executed locally, but may not be confirmed by the network yet         ] 
```

这里将testd::hi action的手续费设置为 0.05 EOS， 注意后面的三个零。

注意设置手续费需要合约账户权限。

setfee 命令使用方式如下：

```
Set Fee need to action
Usage: ./cleos set setfee [OPTIONS] account action fee cpu_limit net_limit ram_limit

Positionals:
  account TEXT                The account to set the Fee for (required)
  action TEXT                 The action to set the Fee for (required)
  fee TEXT                    The Fee to set to action (required)
  cpu_limit UINT              The cpu max use limit to set to action (required)
  net_limit UINT              The net max use limit to set to action (required)
  ram_limit UINT              The ram max use limit to set to action (required)
```

其中

- account 合约账户名
- action 要设置手续费的action名
- fee 手续费，只接受eos
- cpu_limit 每次执行合约使用的cpu资源上限，单位为us， 对于用户合约来说此项为0
- net_limit 每次执行合约使用的net资源上限，单位为byte， 对于用户合约来说此项为0
- ram_limit 每次执行合约使用的ram资源上限，单位为byte， 对于用户合约来说此项为0

开发者需set code 、set abi上传合约。

详情可加入‘原力主网dapp开发群’微信群进行咨询。

### 3. 出块速度

对于依赖时间的合约，需要注意的是我们是三秒出一个块。

### 4. 延迟交易暂未开放

暂定下次主网升级开放支持
