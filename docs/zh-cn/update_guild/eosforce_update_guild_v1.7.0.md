# EOSForce v1.7.0 更新说明

更新分为四步: 

1. 首先通过bp多签为抵押合约创建超级节点押金类型, 并设置更新后超级节点换届延期区块高度
2. 通过bp多签更新系统合约
3. 等待一段时间给bp缴纳押金之后, 通过bp多签开启节点惩罚机制
4. 在超级节点换届延期区块高度之后恢复换届, 这一步是自动执行的

其中1、2步完成之后定期投票功能将会激活, 用户可以通过命令行定期投票.


## 1. bp多签为抵押合约创建超级节点押金类型

可以通过命令review, 查看多签内容:

```bash
./cleos https://w3.eosforce.cn:443 multisig review eosio.pledge addblock.out
{
  "proposer": "eosio.pledge",
  "proposal_name": "addblock.out",
  "transaction_id": "2ed951e2ae5b0e89a4b0833d7c86e8fb7287e3c21ef4dfb76b9d9019bfe701f2",
  "packed_transaction": "c61e625d0000000000000000000001a05852b102ea305500000040559f5332010000000000ea305500000000a8ed3232360000c89a0288683c0000000000ea30550000000000ea3055102700000000000004454f53000000000d61646420626c6f636b206f757400983a00000000000004454f5300000000",
  "transaction": {
    "expiration": "2019-08-25T05:38:14",
    "ref_block_num": 0,
    "ref_block_prefix": 0,
    "max_net_usage_words": 0,
    "max_cpu_usage_ms": 0,
    "delay_sec": 0,
    "context_free_actions": [],
    "actions": [{
        "account": "eosio.pledge",
        "name": "addtype",
        "authorization": [{
            "actor": "eosio",
            "permission": "active"
          }
        ],
        "data": {
          "pledge_name": "block.out",
          "deduction_account": "eosio",
          "ram_payer": "eosio",
          "quantity": "1.0000 EOS",
          "memo": "add block out"
        },
        "hex_data": "0000c89a0288683c0000000000ea30550000000000ea3055102700000000000004454f53000000000d61646420626c6f636b206f7574"
      }
    ],
    "transaction_extensions": [],
    "fee": "1.5000 EOS"
  }
}
```

节点批准(这里账户是`biosbpb`):

```bash
./cleos https://w3.eosforce.cn:443 multisig approve eosio.pledge addblock.out '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
```

## 2. bp多签设置更新后超级节点换届延期区块高度

为了保证过渡到定期投票整个主网bp过渡的连续性, 本次更新将会延迟一段时间的主网bp换届, 目前确定的延迟截止区块高度为11975000, 大致是当前时间的两天后.

可以通过命令review, 查看多签内容:

```bash
./cleos https://w3.eosforce.cn:443 multisig review force.msig update.bp
{
  "proposer": "force.msig",
  "proposal_name": "update.bp",
  "transaction_id": "4a6a654575fe1998d1c90f0e49bcb5e5ee1b4fc6c61fa64bde57a8d81bb72755",
  "packed_transaction": "0a26625d00000000000000000000010000000000ea30550000606e4d8ab2c2010000000000ea305500000000a8ed3232280000a807a86c52d558b9b600000000000000000000000000000000000000000004454f5300000000000a0000000000000004454f5300000000",
  "transaction": {
    "expiration": "2019-08-25T06:09:14",
    "ref_block_num": 0,
    "ref_block_prefix": 0,
    "max_net_usage_words": 0,
    "max_cpu_usage_ms": 0,
    "delay_sec": 0,
    "context_free_actions": [],
    "actions": [{
        "account": "eosio",
        "name": "setconfig",
        "authorization": [{
            "actor": "eosio",
            "permission": "active"
          }
        ],
        "data": {
          "typ": "update.bp",
          "num": 11975000,
          "key": "",
          "fee": "0.0000 EOS"
        },
        "hex_data": "0000a807a86c52d558b9b600000000000000000000000000000000000000000004454f5300000000"
      }
    ],
    "transaction_extensions": [],
    "fee": "0.0010 EOS"
  }
}
```

节点批准(这里账户是`biosbpb`):

```bash
./cleos https://w3.eosforce.cn:443 multisig approve force.msig update.bp '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
```

## 3. 通过bp多签更新系统合约

之后需要通过多签执行更新系统合约, 这里code和abi需要分别批准:

```bash
./cleos https://w3.eosforce.cn:443 multisig review force.msig p.upsyscode
./cleos https://w3.eosforce.cn:443 multisig review force.msig p.upsysabi
```

节点批准(这里账户是`biosbpb`):

```bash
./cleos https://w3.eosforce.cn:443 multisig approve force.msig p.upsyscode '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
./cleos https://w3.eosforce.cn:443 multisig approve force.msig p.upsysabi '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
```

## 4. 等待一段时间给bp缴纳押金之后, 通过bp多签开启节点惩罚机制

这一步需要等待一段时间给bp来确认押金情况, 目前无法执行:

节点批准(这里账户是`biosbpb`):

```bash
./cleos https://w3.eosforce.cn:443 multisig approve force.msig c.bppunish '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
```
