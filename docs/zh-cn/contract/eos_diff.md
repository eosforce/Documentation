# 合约开发注意事项

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

### 2. 合约审核与手续费机制。

免除了内存、CPU、NET的购买与使用操作。仅对action设置资源限制。
合约内对multi_index表emplace()\modify()操作的资源使用者payer参数，可以设置为0或_self本合约账号。

目前开发者开发完成智能合约后，需提交至eos原力开发者委员会进行审核。委员会对合约进行安全性审核，并对每个action通过评估设置合理手续费与资源限制后，此合约才能正常使用。

开发者需set code 、set abi上传合约。

详情可加入‘原力主网dapp开发群’微信群进行咨询。

### 3. 出块速度

对于依赖时间的合约，需要注意的是我们是三秒出一个块。

### 4. 延迟交易暂未开放

暂定下次主网升级开放支持
