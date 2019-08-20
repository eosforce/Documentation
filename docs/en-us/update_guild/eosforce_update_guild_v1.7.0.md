# EOSForce v1.7.0 Update Doc

The update is divided into four steps:

1. First create a block producer deposit type for the eosio.pledge contract through bp multi-sign, and set the block num of the block producer change after the update.
2. Update eosio.system contract through bp multi-sign
3. After waiting for a period of time to pay the deposit for bps, open the block producers penalty mechanism through bp multi-sign
4. After the `update.bp` block height, resume the block producer change, this step will automatically executed.

After the completion of steps 1, 2, the fix-time voting function will be activated, and the user can vote regularly through the command line, the wallet client will update after update success.

## 1. create a block producer deposit type for the eosio.pledge

You can check the multi-sign content by command review:

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

block producer approval (here the account is `biosbpb`) :

```bash
./cleos https://w3.eosforce.cn:443 multisig approve eosio.pledge addblock.out '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
```

## 2. set `update.bp`

In order to ensure the continuity of the change of bps, this update will delay the change of the bps for a period of time.
The current determined delay block height is 11975000, roughly two days after the current time.

You can check the multi-sign content by command review:

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

block producer approval (here the account is `biosbpb`) :

```bash
./cleos https://w3.eosforce.cn:443 multisig approve force.msig update.bp '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
```

## 3. Update eosio.system contract through bp multi-sign

Here the code and abi updates need to be approved separately:

You can check the multi-sign content by command review:

```bash
./cleos https://w3.eosforce.cn:443 multisig review force.msig p.upsyscode
./cleos https://w3.eosforce.cn:443 multisig review force.msig p.upsysabi
```

block producer approval (here the account is `biosbpb`) :

```bash
./cleos https://w3.eosforce.cn:443 multisig approve force.msig p.upsyscode '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
./cleos https://w3.eosforce.cn:443 multisig approve force.msig p.upsysabi '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
```

## 4.  After waiting for a period of time to pay the deposit for bps, open the block producers penalty mechanism through bp multi-sign

This step needs to wait for a period of time to bp to confirm the deposit state, currently unable to execute:

block producer approval (here the account is `biosbpb`) :

```bash
./cleos https://w3.eosforce.cn:443 multisig approve force.msig c.bppunish '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
```


