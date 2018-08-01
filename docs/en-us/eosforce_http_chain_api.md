# chain api

- Protocol: http
- Bash Path: /v1/chain/
- Request Method: POST

## /get_info
Returns an object containing various details about the blockchain.

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
Returns an object containing various details about a specific block on the blockchain.

http://127.0.0.1:8888/v1/chain/get_block

BODY PARAMS:
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

BODY PARAMS:
- block_num_or_id : string  (REQUIRED)    Provide a block number or a block id


## /get_account
Returns an object containing various details about a specific account on the blockchain.

BODY PARAMS:
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
Returns an object containing rows from the specified table.

BODY PARAMS	:
- scope : string REQUIRED Provide the account name
- code : string REQUIRED Provide the smart contract name
- table : string REQUIRED Provide the table name
- json : string REQUIRED Provide true or false
- lower_bound : int32, Provide the lower bound
- upper_bound : int32, Provide the upper bound
- limit : int32, Provide the limit

## /get_code
Returns an object containing various details about a specific smart contract on the blockchain.

BODY PARAMS	:
- account_name : string (REQUIRED) Provide a smart contract account name
- code_as_wasm : boolean

## /get_abi

BODY PARAMS	:
- account_name : string, name of account to retrieve ABI for

## ~~/get_producers~~
invalid in eosforce

BODY PARAMS	 :
- limit : string, total number of producers to retrieve
- lower_bound : string
- json : boolean, return result in JSON format?

## /get_currency_balance
Query token currency balance.

- code string, eg: eosio.token
- account : string ,account to get balance of
- symbol : string, symbol to query
  
## /get_currency_stats
Query token currency stats.

BODY PARAMS:
- code string, eg: eosio.token
- symbol : string, symbol to query

## /get_required_keys
Returns the required keys needed to sign a transaction.

BODY PARAMS:
- transaction : json, REQUIRED  Provide the transaction object
- available_keys : array of strings , REQUIRED Provide the available keys

## /get_required_fee

get required fee of transaction, additional in eosforce.

BODY PARAMS:
- transaction : json, the transaction that will be pushed

RETURN :
{"required_fee":"0.1000 EOS"}

## /abi_json_to_bin
Serializes json to binary hex. The resulting binary hex is usually used for the data field in push_transaction.
support only abi of account eosio and eosio.token by far.

BODY PARAMS	 :
- code :  string, Provide the account name.
- action : string, Provide the action arguments
- args : json Provide the json arguments

## /abi_bin_to_json
Serializes binary hex to json.

BODY PARAMS	
- code string REQUIRED Provide the smart contract account name
- action : string REQUIRED Provide the action name
- binargs : string REQUIRED Provide the binary arguments
  
## /push_transaction

BODY PARAMS	:
- expiration : string, REQUIRED, Provide the experation
- ref_block_num : string, REQUIRED Provide the reference block number. See note below.
- ref_block_prefix : string, REQUIRED, Provide the reference block prefix number. See note below.
- max_net_usage_words : int64
- max_cpu_usage_ms : int32
- delay_sec : int32
- context_free_actions : array
- actions : array[action]
- transaction_extensions : string
- signatures : array of strings, Provide the signatures for the transaction
- context_free_data : array of strings
- fee : string , eg: '1.0000 EOS'

> Note: 
> 
> The ref_block_num and ref_block_prefix here are provided as a result of /v1/chain/get_block of the last_irreversible_block. The last_irreversible_block can be found by calling /v1/chain/get_info. 
> You also need to use /v1/wallet/sign_transaction to get the right signature.
>
>eosforce add fee in the transaction.

## /push_transactions
This method expects a transactions in JSON format and will attempt to apply it to the blockchain. This method push multiple transactions at once.
BODY PARAMS	:

- body : json ,REQUIRED, Provide the authorizations for the transaction
