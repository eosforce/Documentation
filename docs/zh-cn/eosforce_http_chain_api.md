# 链 api

- 协议: http
- 基础路径: /v1/chain/
- 请求方法: POST

各接口路径:

## /get_info

获取链相关信息

http://127.0.0.1:8888/v1/chain/get_info

```json
{
    "server_version": "3ef7e981",
    "chain_id": "bd61ae3a031e8ef2f97ee3b0e62776d6d30d4833c8f7c1645c657b149151004b",
    "head_block_num": 1148120,
    "last_irreversible_block_num": 1148089,
    "last_irreversible_block_id": "001184b9980b44d49c0adeaf2801ec127484c7fc2db95076c7a128b14cb7bd65",
    "head_block_id": "001184d81bba61f342d036567039ac45f62d6b865066a5aee417905215967de8",
    "head_block_time": "2018-08-01T03:51:00",
    "head_block_producer": "eosgod",
    "virtual_block_cpu_limit": 200000000,
    "virtual_block_net_limit": 1048576000,
    "block_cpu_limit": 199900,
    "block_net_limit": 1048576
}
```

## /get_block

获取某一区块信息

http://127.0.0.1:8888/v1/chain/get_block

参数:
- block_num_or_id : string  (REQUIRED)    Provide a block number or a block id

```json
{
    "timestamp": "2018-06-22T13:03:12.000",
    "producer": "biosbpk",
    "confirmed": 22,
    "previous": "000028bb191717a6d982ab6a41b5d290d278723b6fd47d846fd4d4cebe1e73a0",
    "transaction_mroot": "0000000000000000000000000000000000000000000000000000000000000000",
    "action_mroot": "7f6653a8839fb640f4dd40cce4152d2b5ec9796f22b8533a3a41f14201a79014",
    "schedule_version": 0,
    "new_producers": null,
    "header_extensions": [],
    "producer_signature": "SIG_K1_KAAMUd2C3exCkujQ8snVkncm6V1sigiLHPn1PqjMRrxNZpN8Vw4HV7cpDhnrFH2aHBDqPY5aoCcysaAEKeD3nR7jm8Jx6v",
    "transactions": [],
    "block_extensions": [],
    "id": "000028bc9591c14f676ef01b2c67210edf51d43e8eda8374a96fa103c9c38892",
    "block_num": 10428,
    "ref_block_prefix": 468741735
}
```

## /get_block_header_state
获取区块头状态

参数:
- block_num_or_id : string  (REQUIRED)    Provide a block number or a block id
```json
{
    "id": "0011965e344083a30c7e7dd7fabed3b424090cd3be2ecc756a10ffbd541be4e2",
    "block_num": 1152606,
    "header": {
        "timestamp": "2018-08-01T07:35:39.000",
        "producer": "everest",
        "confirmed": 22,
        "previous": "0011965d8e1274c30ebdf77486958f457206c8d7640bff29f1a1beaddfc87720",
        "transaction_mroot": "bbc406d13bd15bd4a0977c9a279d529b767735a259250dec36ce836f10e9bcaf",
        "action_mroot": "7439446bf7a0f31120af15d5e3a8ec091eece212874cd3dbb9eb5b7ee7814600",
        "schedule_version": 212,
        "header_extensions": [],
        "producer_signature": "SIG_K1_KaRk1APUm8szYCQkpqT9wV96RbMAL9UAjbQ4TWrh1XCkyEEzT82JqZsQ6hp2q76HxoMZoUe8wzV7vtLSP6mdpSKGevWJM5"
    },
    "dpos_proposed_irreversible_blocknum": 1152591,
    "dpos_irreversible_blocknum": 1152575,
    "bft_irreversible_blocknum": 0,
    "pending_schedule_lib_num": 1152232,
    "pending_schedule_hash": "4e63b8afd0e63ca110367d13330c3ea69f37bab2740b6b975b1d13d110904ccf",
    "pending_schedule": {
        "version": 212,
        "producers": []
    },
    "active_schedule": {
        "version": 212,
        "producers": [
            {
                "producer_name": "ccbc",
                "block_signing_key": "EOS7p1Rc2NG2KTzxBdSLCBFCBULeySsMCAnTRWaZtYmaGwDUrpr8o"
            },
            {
                "producer_name": "cindydaily",
                "block_signing_key": "EOS5ZS4234GvHSY7JXooSU2hLRtNipoq2MAXvdXPvnGr91XNuKSov"
            },
            {
                "producer_name": "coinlord.one",
                "block_signing_key": "EOS7N1TzYeRWSBA4mybvTBQw6coWwgrj3GoUQZLPTtAKbaFnoJSAd"
            }
        ]
    },
    "blockroot_merkle": {
        "_active_nodes": [
            "0011965d8e1274c30ebdf77486958f457206c8d7640bff29f1a1beaddfc87720",
            "e8a05f0b477956f4f561cc6f66b2d6fdfcc264de508edb905cf8fa29d8fb225e",
            "3a498916dbeb39a7ed15b230e1a0100e803412b0d922e2e7213b21a10ac775ef"
        ],
        "_node_count": 1152605
    },
    "producer_to_last_produced": [
        [ "ccbc", 1152577 ], [ "cindydaily", 1152578 ], [ "coinlord.one", 1152579 ]
    ],
    "producer_to_last_implied_irb": [
        [ "ccbc", 1152577 ], [ "cindydaily", 1152578 ], [ "coinlord.one", 1152579 ]
    ],
    "block_signing_key": "EOS7PEgE82LpXqvWJLjF52hLW5mqE73ZNXPorXgoj9dgzv9Q7HRfN",
    "confirm_count": [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15 ],
    "confirmations": []
}
```

## /get_account

获取账号信息

参数:
- account_name : string (REQUIRED) Provide an account name

```json
{
    "account_name": "eosio",
    "privileged": true,
    "last_code_update": "2018-06-22T02:00:00.000",
    "created": "2018-06-22T02:00:00.000",
    "ram_quota": -1,
    "net_weight": -1,
    "cpu_weight": -1,
    "net_limit": {
        "used": -1,
        "available": -1,
        "max": -1
    },
    "cpu_limit": {
        "used": -1,
        "available": -1,
        "max": -1
    },
    "ram_usage": 2724,
    "permissions": [
        {
            "perm_name": "active",
            "parent": "owner",
            "required_auth": {
                "threshold": 1,
                "keys": [
                    {
                        "key": "EOS1111111111111111111111111111111114T1Anm",
                        "weight": 1
                    }
                ],
                "accounts": [],
                "waits": []
            }
        },
        {
            "perm_name": "owner",
            "parent": "",
            "required_auth": {
                "threshold": 1,
                "keys": [
                    {
                        "key": "EOS1111111111111111111111111111111114T1Anm",
                        "weight": 1
                    }
                ],
                "accounts": [],
                "waits": []
            }
        }
    ],
    "total_resources": null,
    "delegated_bandwidth": null,
    "voter_info": null
}
```
## /get_table_rows

查询表数据信息

参数	:
- scope : string REQUIRED Provide the account name
- code : string REQUIRED Provide the smart contract name
- table : string REQUIRED Provide the table name
- json : string REQUIRED Provide true or false
- lower_bound : int32, Provide the lower bound
- upper_bound : int32, Provide the upper bound
- limit : int32, Provide the limit

## /get_code

获取智能合约

参数	:
- account_name : string (REQUIRED) Provide a smart contract account name
- code_as_wasm : boolean

## /get_abi
获取abi信息

参数	:
- account_name : string, name of account to retrieve ABI for

## ~~/get_producers~~
eosforce无效

参数	 :
- limit : string, total number of producers to retrieve
- lower_bound : string
- json : boolean, return result in JSON format?

## /get_currency_balance
查询代币余额

- code string, eg: eosio.token
- account : string ,account to get balance of
- symbol : string, symbol to query
  
## /get_currency_stats
查询代币状态

参数:
- code string, eg: eosio.token
- symbol : string, symbol to query

## /get_required_keys

获取执行一个交易签名所需的公钥

参数:
- transaction : json, REQUIRED  Provide the transaction object
- available_keys : array of strings , REQUIRED Provide the available keys

## /get_required_fee

获取交易所需的手续费

参数:
- transaction : json, the transaction that will be pushed

RETURN :
{"required_fee":"0.1000 EOS"}

## /abi_json_to_bin

json序列化，序列化结果用于push_transaction的data字段。
目前仅支持 eosio and eosio.token 账号的abi， 之后会支持eosio.bios账号，包括newaccount等系统action

参数	 :
- code :  string, Provide the account name.
- action : string, Provide the action arguments
- args : json Provide the json arguments

## /abi_bin_to_json

反序列化成json

参数	
- code string REQUIRED Provide the smart contract account name
- action : string REQUIRED Provide the action name
- binargs : string REQUIRED Provide the binary arguments
  
## /push_transaction

推送交易

参数	:
- expiration : string, REQUIRED, Provide the experation
- ref_block_num : string, REQUIRED Provide the reference block number. See note below.
- ref_block_prefix : string, REQUIRED, Provide the reference block prefix number. See note below.
- max_net_usage_words : int64
- max_cpu_usage_ms : int32
- delay_sec : int32
- context_free_actions : array
- actions : array[action]
- transaction_extensions : string
- signatures : array of strings, 签名
- context_free_data : array of strings
- fee : string ,手续费， eg: '1.0000 EOS'

> 注: 
> 
> 通过接口：/v1/chain/get_info 获取 last_irreversible_block，
> 再通过 last_irreversible_block 调接口：/v1/chain/get_block 可获取 ref_block_num， ref_block_prefix。
> 
> eosforce的钱包是在本地执行签名的，也可以调用 /v1/wallet/sign_transaction 获取签名。
>
> eosforce 在交易中增加了手续费字段

## /push_transactions

批量推送交易

参数	:

- body : json ,REQUIRED, Provide the authorizations for the transaction
