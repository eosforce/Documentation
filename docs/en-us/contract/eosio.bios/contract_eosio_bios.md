# eosio.bios 系统合约

通过此合约eos系统可以直接控制其它账户的资源分配，并且可以调用其它需要特殊权限的API。
主要包括账号与权限设置，合约设置，资源分配等

## 合约方法 actions

- [newaccount](zh-cn/contract/eosio.bios/newaccount.md)  : 创建账号 

- [updateauth](zh-cn/contract/eosio.bios/updateauth.md)  : 修改权限

- deleteauth : 删除权限

> 以下eosforce未开放

- setcode : 设置合约code
- setabi : 设置合约abi
- linkauth : 链接权限
- unlinkauth : 取消权限链接
- canceldelay : 
- onerror

- setalimits
- setglimits
- setpriv
- setprods
- reqauth



## 相关源码

- [eosio/chain/eosio_contract.hpp](https://github.com/eosforce/eosforce/blob/master/libraries/chain/include/eosio/chain/eosio_contract.hpp)
- [eosio/chain/eosio_contract.cpp](https://github.com/eosforce/eosforce/blob/master/libraries/chain/eosio_contract.cpp)
- [eosio.bios/eosio.bios.hpp](https://github.com/eosforce/eosforce/blob/master/contracts/eosio.bios/eosio.bios.hpp)

## abi 数据结构
```json
 "types": [{
      "new_type_name": "permission_name",
      "type": "name"
   },{
      "new_type_name": "action_name",
      "type": "name"
   },{
      "new_type_name": "transaction_id_type",
      "type": "checksum256"
   },{
      "new_type_name": "weight_type",
      "type": "uint16"
   }],
"structs": [{
      "name": "permission_level",
      "base": "",
      "fields": [
        {"name":"actor",      "type":"account_name"},
        {"name":"permission", "type":"permission_name"}
      ]
    },{
      "name": "key_weight",
      "base": "",
      "fields": [
        {"name":"key",    "type":"public_key"},
        {"name":"weight", "type":"weight_type"}
      ]
    },{
      "name": "permission_level_weight",
      "base": "",
      "fields": [
        {"name":"permission", "type":"permission_level"},
        {"name":"weight",     "type":"weight_type"}
      ]
    },{
      "name": "wait_weight",
      "base": "",
      "fields": [
        {"name":"wait_sec", "type":"uint32"},
        {"name":"weight",   "type":"weight_type"}
      ]
    },{
      "name": "authority",
      "base": "",
      "fields": [
        {"name":"threshold", "type":"uint32"},
        {"name":"keys",      "type":"key_weight[]"},
        {"name":"accounts",  "type":"permission_level_weight[]"},
        {"name":"waits",     "type":"wait_weight[]"}
      ]
    },{
      "name": "newaccount",
      "base": "",
      "fields": [
        {"name":"creator", "type":"account_name"},
        {"name":"name",    "type":"account_name"},
        {"name":"owner",   "type":"authority"},
        {"name":"active",  "type":"authority"}
      ]
    },{
      "name": "setcode",
      "base": "",
      "fields": [
        {"name":"account",   "type":"account_name"},
        {"name":"vmtype",    "type":"uint8"},
        {"name":"vmversion", "type":"uint8"},
        {"name":"code",      "type":"bytes"}
      ]
    },{
      "name": "setabi",
      "base": "",
      "fields": [
        {"name":"account", "type":"account_name"},
        {"name":"abi",     "type":"bytes"}
      ]
    },{
      "name": "updateauth",
      "base": "",
      "fields": [
        {"name":"account",    "type":"account_name"},
        {"name":"permission", "type":"permission_name"},
        {"name":"parent",     "type":"permission_name"},
        {"name":"auth",       "type":"authority"}
      ]
    },{
      "name": "deleteauth",
      "base": "",
      "fields": [
        {"name":"account",    "type":"account_name"},
        {"name":"permission", "type":"permission_name"}
      ]
    },{
      "name": "linkauth",
      "base": "",
      "fields": [
        {"name":"account",     "type":"account_name"},
        {"name":"code",        "type":"account_name"},
        {"name":"type",        "type":"action_name"},
        {"name":"requirement", "type":"permission_name"}
      ]
    },{
      "name": "unlinkauth",
      "base": "",
      "fields": [
        {"name":"account",     "type":"account_name"},
        {"name":"code",        "type":"account_name"},
        {"name":"type",        "type":"action_name"}
      ]
    },{
      "name": "canceldelay",
      "base": "",
      "fields": [
        {"name":"canceling_auth", "type":"permission_level"},
        {"name":"trx_id",         "type":"transaction_id_type"}
      ]
    },{
      "name": "onerror",
      "base": "",
      "fields": [
        {"name":"sender_id", "type":"uint128"},
        {"name":"sent_trx",  "type":"bytes"}
      ]
    },{
      "name": "set_account_limits",
      "base": "",
      "fields": [
        {"name":"account",    "type":"account_name"},
        {"name":"ram_bytes",  "type":"int64"},
        {"name":"net_weight", "type":"int64"},
        {"name":"cpu_weight", "type":"int64"}
      ]
    },{
        "name": "setpriv",
        "base": "",
        "fields": [
          {"name":"account",    "type":"account_name"},
          {"name":"is_priv",    "type":"int8"}
        ]
      },{
      "name": "set_global_limits",
      "base": "",
      "fields": [
        {"name":"cpu_usec_per_period",    "type":"int64"}
      ]
    },{
      "name": "producer_key",
      "base": "",
      "fields": [
        {"name":"producer_name",      "type":"account_name"},
        {"name":"block_signing_key",  "type":"public_key"}
      ]
    },{
      "name": "set_producers",
      "base": "",
      "fields": [
        {"name":"schedule",   "type":"producer_key[]"}
      ]
    },{
      "name": "require_auth",
      "base": "",
      "fields": [
        {"name":"from", "type":"account_name"}
      ]
    }]
```
