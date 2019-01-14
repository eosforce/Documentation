# eosio.token 

## 1. Action

### 1.1 create token

create token symbol

Parameters：
- issuer : type account_name, issuer

- maximum_supply :  type asset, token and maximum supply

> asset 'ammount token symbol' eg:'100 RMB'
  
### 1.2 issue token

issue certain amounts of token to specified account

Parameters：:
- to : type account_name
- quantity : type asset
- memo : type string

### 1.3 transfer token

Parameters：:
- from : 类型 account_name
- to : 类型 account_name
- quantity : 类型 asset
- memo : 类型 string

## 2. table

### 2.1 accounts
token account table

primary key: currency

field: balance 


### 2.2 stat
stat table

primary key:   currency 

fields:
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