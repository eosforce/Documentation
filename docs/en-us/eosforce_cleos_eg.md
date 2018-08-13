# 常用功能cleos命令

### 1. 获取账号余额:

```shell
./cleos get table eosio eosio accounts -k 账户名
```

> 从系统合约accounts表中读取

返回：
```json
{
  "rows": [{
      "name": "b1",
      "available": "103241.0000 EOS"
    }
  ],
  "more": false
}
```

### 2. 通过公钥获取其所有账号

```shell
./cleos get accounts 公钥
```

返回：
```json
{
  "account_names": [
    "alice",
    "bob"
  ]
}
```
### 3. 获取超级节点排行

```shell
 ./cleos get table eosio eosio schedules -k 第几届
```
```json
{
  "rows": [{
      "version": 2,
      "block_height": 4164,
      "producers": [{
          "bpname": "biosbpc",
          "amount": 0
        },{
          "bpname": "biosbpd",
          "amount": 0
        },{
          "bpname": "biosbpe",
          "amount": 6210
        }
      ]
    }
  ],
  "more": false
}
```
### 4. 获取超级节点信息
```shell
./cleos get table eosio eosio bps
./cleos get table eosio eosio bps -k 节点名
```
```json
{
  "rows": [{
      "name": "biosbpi",
      "block_signing_key": "EOS5sWyyriwYhg4iA6K8FxyiGwaCBYoUy2MAQHLEwo4AcdR31hXv9",
      "commission_rate": 1000,
      "total_staked": 10001,
      "rewards_pool": "52440.9600 EOS",
      "total_voteage": 258030000,
      "voteage_update_height": 25887,
      "url": "http://eosforce.io",
      "emergency": 0
    }
  ],
  "more": false
}
```

### 5. 获取投票信息
```shell
./cleos table eosio 账户名 votes
```
```json
{
  "rows": [{
      "bpname": "biosbpa",
      "staked": "33.0000 EOS",
      "voteage": 0,
      "voteage_update_height": 14238,
      "unstaking": "0.0000 EOS",
      "unstake_height": 14238
    },{
      "bpname": "biosbpi",
      "staked": "1.0000 EOS",
      "voteage": 0,
      "voteage_update_height": 14258,
      "unstaking": "0.0000 EOS",
      "unstake_height": 14258
    }
  ],
  "more": false
}
```

### 6. 转账

```shell
./cleos push action eosio transfer '{"from":"alice","to":"bob", "quantity":"1000.0000 EOS", "memo":"hello"}' -p alice@active
```

> 提交System系统合约转账交易

### 7. 投票

```shell
./cleos push action eosio vote '{"voter":"alice","bpname":"bob","stake":"10.0000 EOS"}' -p alice@active
```
> 提交System系统合约投票交易，stake为对bp的当前投票数

### 8. 领取分红

```shell
./cleos push action eosio claim '{"voter":"alice","bpname":"bob"}'  -p alice@active
```

> 提交System系统合约分红交易

### 9. 解冻

```shell
./cleos push action eosio unfreeze '{"voter":"alice","bpname":"bob"}'  -p alice@active
```

> 提交System系统合约解冻交易 unfreeze


> ./cleos push action 为cleos客户端封装好的包含一个action的交易transaction请求。eosio为System系统合约的创建账户。-p指定当前交易的执行账户与权限。