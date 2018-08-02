# eosio.token 代币合约

## 1. Action

### 1.1 create 创建代币

创建代币符号

参数：
- issuer : 类型 account_name, 发行人
- maximum_supply :  类型 asset, 代币及最大供应量

> asset '数量 代币符号' eg:'100 RMB'
  
### 1.2 issue 发行代币

发行一定数量代币到指定账户

参数:
- to : 类型 account_name
- quantity : 类型 asset
- memo : 类型 string

### 1.3 transfer 代币转账

代币转账操作

参数:
- from : 类型 account_name
- to : 类型 account_name
- quantity : 类型 asset
- memo : 类型 string

## 2. 表

### 2.1 accounts
代币账户表

主键: currency

字段: balance 余额


### 2.2 stat
代码状态表

主键:   currency : 代币符号

字段:
        supply : asset
        max_supply : asset
        issuer : account_name

## ABI
```json
{
   "version": "eosio::abi/1.0",
   "types": [{
      "new_type_name": "account_name",
      "type": "name"
   }],
  "structs": [{
      "name": "transfer",
      "base": "",
      "fields": [
        {"name":"from", "type":"account_name"},
        {"name":"to", "type":"account_name"},
        {"name":"quantity", "type":"asset"},
        {"name":"memo", "type":"string"}
      ]
    },{
     "name": "create",
     "base": "",
     "fields": [
        {"name":"issuer", "type":"account_name"},
        {"name":"maximum_supply", "type":"asset"}
     ]
  },{
     "name": "issue",
     "base": "",
     "fields": [
        {"name":"to", "type":"account_name"},
        {"name":"quantity", "type":"asset"},
        {"name":"memo", "type":"string"}
     ]
  },{
      "name": "account",
      "base": "",
      "fields": [
        {"name":"balance", "type":"asset"}
      ]
    },{
      "name": "currency_stats",
      "base": "",
      "fields": [
        {"name":"supply", "type":"asset"},
        {"name":"max_supply", "type":"asset"},
        {"name":"issuer", "type":"account_name"}
      ]
    }
  ],
  "actions": [{
      "name": "transfer",
      "type": "transfer",
      "ricardian_contract": ""
    },{
      "name": "issue",
      "type": "issue",
      "ricardian_contract": ""
    }, {
      "name": "create",
      "type": "create",
      "ricardian_contract": ""
    }

  ],
  "tables": [{
      "name": "accounts",
      "type": "account",
      "index_type": "i64",
      "key_names" : ["currency"],
      "key_types" : ["uint64"]
    },{
      "name": "stat",
      "type": "currency_stats",
      "index_type": "i64",
      "key_names" : ["currency"],
      "key_types" : ["uint64"]
    }
  ],
  "ricardian_clauses": [],
  "abi_extensions": []
}

```