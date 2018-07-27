# 多重签名

## 功能概述

eos账号权限体系支持了多重签名功能，一个账户创建时默认有owner、active两个权限，每个权限的开启阈值为1，由公钥key组成，每个key的权重为1。当key权重和大于等于权限阈值时，该权限才能被成功授权。默认的配置就需要一个签名就可以授权来做某些行为。因为每个key的权重>=开启阈值。

```bash
./cleos get account user1
permissions:
     owner     1:    1 EOS8T5BG84e9L9BdUFxg7a5xiTfbE51zBPt4mKkesuhm4ZWx7jAe5
        active     1:    1 EOS8T5BG84e9L9BdUFxg7a5xiTfbE51zBPt4mKkesuhm4ZWx7jAe5
```

设置账号权限有多个key，并且权限开启阈值大于单个key的权重时，就需要多个key签名同时授权才能生效，即多重签名。

示例：

设置ssh3账号active权限阈值为2，accounts为alice何bob的active权限，权重各为1。这样ssh3账号就只能通过alice与bob两个账号授权才能进行active权限的操作了。

```bash
./cleos set account permission ssh3 active '{ "threshold": 2, "keys": [], "accounts":[ { "permission": { "actor": "alice", "permission": "active" }, "weight": 1 }, { "permission": { "actor": "bob", "permission": "active" }, "weight": 1 } ] }' owner

./cleos get account ssh3
permissions:
     owner     1:    1 EOS8T5BG84e9L9BdUFxg7a5xiTfbE51zBPt4mKkesuhm4ZWx7jAe5
        active     2:    1 alice@active, 1 bob@active,
```

> 账号权限可以设置为公钥，也可以是其他账号。

## eosio.msig 多签合约

当多签账号的key分布在不同钱包中，就需要一种异步协同的机制实现多签授权执行交易。eos提供了**eosio.msig**合约，合约提供了一个同步侧通道，数据在这个通道中传输并签名。 Eosio.msig是一个对用户更加友好的方式，异步提出、批准并最终发布多方同意的交易。

**eosio.msig**合约 action：

1. propose 提议
2. approve 批准 
3. unapprove 取消批准
4. exec 执行
5. cancel 取消提议

## 多签执行流程

### 1. 创建提议

```bash
./cleos multisig propose pab1 '[{"actor":"alice","permission":"active"},{"actor":"bob","permission":"active"}]' '[{"actor":"ssh3","permission":"active"}]' eosio transfer '{"from":"ssh3","to":"ssh","quantity":"66.0000 EOS","memo":"msig transfer"}' ssh2
```

cleos multisig propose 命令参数解释：

- proposal_name 议题名
- requested_permissions 请求权限 
- trx_permissions 执行的权限
- contract 执行的合约账号
- action 执行的动作
- data 执行的数据
- proposer提议人

### 2. 批准提议

```bash
./cleos multisig approve ssh2 pab1 '{"actor":"alice","permission":"active"}' -p alice@active
./cleos multisig approve ssh2 pab1 '{"actor":"bob","permission":"active"}' -p bob@active
```

cleos multisig approve 命令参数解释：

- proposer提议人
- proposal_name议题名
- permissions权限许可

### 3. 执行提议

```bash
./cleos multisig exec ssh2 pab1 ssh2
```

cleos multisig exec 命令参数解释：

- proposer提议人
- proposal_name议题名
- executer 执行者

## 示例脚本

```bash

#!/bin/bash

##########################################################################
# This is a EOS multi sign demo script.
#
# eosforce.io
#
##########################################################################

echo '---------- this is a msig demo'
echo '---------- unlock wallet'
./cleos wallet unlock

echo '---------- create account : alice and bob, and transfer 0.3 EOS'
./cleos create account ssh alice EOS5x4UDqvUsjWXySFqzboR7GYmwipkXV4sgGpgDRqouzd7NprQ5m
./cleos create account ssh bob EOS6BUSXxqmBBMxnCwFowfFsr8Zi1WRtWXguGzUb9oGGpueMSaJbx
./cleos push action eosio transfer '{"from":"ssh","to":"alice", "quantity":"0.3000 EOS", "memo":""}' -p ssh@active
./cleos push action eosio transfer '{"from":"ssh","to":"bob", "quantity":"0.3000 EOS", "memo":""}' -p ssh@active

echo '---------- set ssh3 msig permisson'
#./cleos set account permission ssh3 active d/my_authority_acc.json owner
./cleos set account permission ssh3 active '{ "threshold": 2, "keys": [], "accounts":[ { "permission": { "actor": "alice", "permission": "active" }, "weight": 1 }, { "permission": { "actor": "bob", "permission": "active" }, "weight": 1 } ] }' owner
./cleos get account ssh3

echo '---------- msig propose : ssh3 transfer ssh 60 EOS (by ssh2)'
./cleos multisig propose -j pab1 '[{"actor":"alice","permission":"active"},{"actor":"bob","permission":"active"}]' '[{"actor":"ssh3","permission":"active"}]' eosio transfer '{"from":"ssh3","to":"ssh","quantity":"66.0000 EOS","memo":"msig transfer"}' ssh2

echo '---------- msig approve: alice and bob'
./cleos multisig approve ssh2 pab1 '{"actor":"alice","permission":"active"}' -p alice@active
./cleos multisig approve ssh2 pab1 '{"actor":"bob","permission":"active"}' -p bob@active

echo '---------- get account balance before msig exe'
./cleos get table eosio eosio accounts -k ssh
./cleos get table eosio eosio accounts -k ssh3

echo '---------- msig exe'
./cleos multisig exec ssh2 pab1 ssh2

echo 'sleep 3 ...'
sleep 1
echo 'sleep 2 ..'
sleep 1
echo 'sleep 1 .'
sleep 1

echo '---------- get result'
./cleos get table eosio eosio accounts -k ssh
./cleos get table eosio eosio accounts -k ssh3

echo '---------- maybe not pack in the block yet, try again later (./cleos get table eosio eosio accounts -k ssh)'


```