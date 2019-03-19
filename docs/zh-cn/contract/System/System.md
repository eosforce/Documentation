# eosforce System 系统合约

## 合约说明

System合约是eosforce系统关键功能的实现合约，替换原eosio.system系统合约。
包括投票、分红等，对eos原有逻辑的修改。
改合约使用eosio账户发布，固定在初始化代码中。

## 合约方法 action

### 1. 转账

```cpp
void transfer( const account_name from, const account_name to, const asset quantity, const string memo );
```

参数：

- from : 转账账号
- to : 收款账号
- quantity : 金额，必须是 EOS资产
- memo : 备注, 必须小于256字节
  
### 2. 设置BP（区块生产者）

```cpp
void updatebp( const account_name bpname, const public_key producer_key,
const uint32_t commission_rate, const std::string & url );
```

添加、修改节点信息

参数：
- bpnbame : 节点账号
- producer_key : 节点公钥
- commission_rate : 分红比列 单位:万分之1, 范围:大于等于1，小于等于10000
- url : 官网链接， 不超过64字节

### 3. 投票

```cpp
void vote( const account_name voter, const account_name bpname, const asset stake );
```

给超级节点投票

参数:

- voter : 投票者
- bpbame : 节点
- statke : 票数（eos金额）

逻辑:
1. 设置用户的投票信息：
投票数大于当前票数，增加投票；
投票数小于当前票数，撤回投票；
2. 根据投票数减少相应用户余额
3. 增加节点的总票数， 结算节点当前总票龄

{
        "name": "revote",
        "base": "",
        "fields": [
          {"name":"voter",     "type":"account_name"},
          {"name":"frombp",    "type":"account_name"},
          {"name":"tobp",      "type":"account_name"},
          {"name":"restake",   "type":"asset"}
        ]
    },

### 4. 更换投票

**更换投票**可以使用户在不冻结用户Token的情况下更换用户所投的节点.

```cpp
void revote( const account_name voter, 
             const account_name frombp, 
             const account_name tobp, 
             const asset restake ) {
```
给超级节点投票

参数:

- voter : 投票者
- frombp : 原来所投节点
- tobp : 换投节点
- restake : 更换票数（eos金额）

示例:

假设 `testa` 用户投了 `biosbpa` 节点 `5000.0000 EOS`, 
此时用户希望改投 `biosbpb` 节点 `2000.0000 EOS`, 则可以执行:

```bash
cleos push action eosio revote '{"voter":"testa","frombp":"biosbpa","tobp":"biosbpb","restake":"2000.0000 EOS"}' -p testa
```

在执行之后用户投给`biosbpa`节点的Token为`3000.0000 EOS`,而投给`biosbpb`节点的Token增加了`2000.0000 EOS`.

### 5. 解冻

```cpp
void unfreeze( const account_name voter, const account_name bpname );
```

    撤回投票要冻结3天(根据块数确定，每3秒一块)，3天后可执行解冻。

逻辑：

将撤回票数 加到可用余额中。清空撤回票数。


### 6. 领取分红

```cpp
void claim( const account_name voter, const account_name bpname );
```

参数:

- voter : 投票者
- bpname : 节点名

逻辑:
1. 计算投票账号最新票龄：上次票龄+票数*(当前块高度-上次结算时块高度)
2. 计算节点的总票龄：上次票龄+票数*(当前块高度-节点上次结算时块高度)
3. 计算分红数：节点奖池总数*投票者总票龄/节点总票龄
4. 增加投票账号余额 +=分红数
5. 投票信息票龄清零，更新结算块高度
6. 减少节点奖池 -=分红数，总票龄减少账号票龄，更新计算块高度


### 7. 出块回调函数    (系统内部使用)

```cpp
void onblock(…const account_name bpname,const uint32_t schedule_version);
```

参数:

- bpname : 节点名
- schedule_version : 节点排行换届版本
  
逻辑:
1. 出块节点bpname，producer.amount出块数加1
2. 每个出块奖励9个EOS，按分红比例，将奖励加入出块节点余额，并将剩余分红加入出块节点的奖池中。
3. 每个出块将奖励1个EOS给b1账号(block.one)
4. 每100个块(即5分钟)更新一次超级节点排行. 刷新选举的23个超级节点，遍历所有bp查前23个，调用wasm接口set_proposed_producers重置BP,修改数据库数据

### 8. 设置链紧急状态
```cpp
void setemergency( const account_name bpname, const bool emergency );
```

超过三分之二的节点设置紧急状态，则链将处于紧急状态。将紧急停用所有功能。
参数: 
- bpname : 节点
- emergency : 紧急状态
 
## abi
- [System](https://github.com/eosforce/eosforce/blob/release/contracts/System02/System02.abi)
  
```json
{
  "version": "eosio::abi/1.0",
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
      "name": "transfer",
      "base": "",
      "fields": [
         {"name":"from",     "type":"account_name"},
         {"name":"to",       "type":"account_name"},
         {"name":"quantity", "type":"asset"},
         {"name":"memo",     "type":"string"}
      ]
    },{
      "name": "updatebp",
      "base": "",
      "fields": [
         {"name":"bpname",            "type":"account_name"},
         {"name":"block_signing_key", "type":"public_key"},
         {"name":"commission_rate",   "type":"uint32"},
         {"name":"url",               "type":"string"}
      ]
    },{
      "name": "unfreeze",
      "base": "",
      "fields": [
      {"name":"voter",      "type":"account_name"},
      {"name":"bpname",     "type":"account_name"}
      ]
    },{
      "name": "claim",
      "base": "",
      "fields": [
        {"name":"voter",      "type":"account_name"},
        {"name":"bpname",     "type":"account_name"}
      ]
    },{
      "name": "account_info",
      "base": "",
      "fields": [
        {"name":"name",         "type":"account_name"},
        {"name":"available",    "type":"asset"}
      ]
    },{
      "name": "vote",
      "base": "",
      "fields": [
        {"name":"voter",     "type":"account_name"},
        {"name":"bpname",    "type":"account_name"},
        {"name":"stake",     "type":"asset"}
      ]
    },{
        "name": "revote",
        "base": "",
        "fields": [
          {"name":"voter",     "type":"account_name"},
          {"name":"frombp",    "type":"account_name"},
          {"name":"tobp",      "type":"account_name"},
          {"name":"restake",   "type":"asset"}
        ]
    },{
      "name": "vote_info",
      "base": "",
      "fields": [
        {"name":"bpname",                "type":"account_name"},
        {"name":"staked",                "type":"asset"},
        {"name":"voteage",               "type":"int64"},
        {"name":"voteage_update_height", "type":"uint32"},
        {"name":"unstaking",             "type":"asset"},
        {"name":"unstake_height",        "type":"uint32"}
      ]
    },{
      "name": "bp_info",
      "base": "",
      "fields": [
        {"name":"name",                 "type":"account_name"},
        {"name":"block_signing_key",    "type":"public_key"},
        {"name":"commission_rate",      "type":"uint32"},
        {"name":"total_staked",         "type":"int64"},
        {"name":"rewards_pool",         "type":"asset"},
        {"name":"total_voteage",        "type":"int64"},
        {"name":"voteage_update_height","type":"uint32"},
        {"name":"url",                  "type":"string"},
        {"name":"emergency",            "type":"bool"}
      ]
    },{
      "name": "setemergency",
      "base": "",
      "fields": [
        {"name":"bpname",    "type":"account_name"},
        {"name":"emergency", "type":"bool"}
      ]
    },{
      "name": "chain_status",
      "base": "",
      "fields": [
        {"name":"name",       "type":"account_name"},
        {"name":"emergency",  "type":"bool"}
      ]
    },{
      "name": "producer",
      "base": "",
      "fields": [
        {"name":"bpname",  "type":"account_name"},
        {"name":"amount",  "type":"uint32"}
      ]
    },{
      "name": "schedule_info",
      "base": "",
      "fields": [
        {"name":"version",     "type":"uint64"},
        {"name":"block_height","type":"uint32"},
        {"name":"producers",   "type":"producer[]"}
      ]
    },{
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
       "name": "setconfig",
       "base": "",
       "fields": [
         {"name":"typ",  "type":"account_name"},
         {"name":"num",  "type":"int64"},
         {"name":"key",  "type":"account_name"},
         {"name":"fee",  "type":"asset"}
       ]
     },{
       "name": "onfee",
       "base": "",
       "fields": [
         {"name":"actor",  "type":"account_name"},
         {"name":"fee",    "type":"asset"},
         {"name":"bpname", "type":"account_name"}
       ]
     },{
       "name": "heartbeat_info",
       "base": "",
       "fields": [
         {"name":"bpname",  "type":"account_name"},
         {"name":"timestamp",    "type":"time_point_sec"}
       ]
     },{
       "name": "removebp",
       "base": "",
       "fields": [
         {"name":"bpname",  "type":"account_name"}
       ]
     }
  ],
  "actions": [{
      "name": "transfer",
      "type": "transfer",
      "ricardian_contract": ""
    },{
      "name": "updatebp",
      "type": "updatebp",
      "ricardian_contract": ""
    },{
      "name": "vote",
      "type": "vote",
      "ricardian_contract": ""
    },{
      "name": "revote",
      "type": "revote",
      "ricardian_contract": ""
    },{
      "name": "unfreeze",
      "type": "unfreeze",
      "ricardian_contract": ""
    },{
      "name": "vote4ram",
      "type": "vote",
      "ricardian_contract": ""
    },{
      "name": "unfreezeram",
      "type": "unfreeze",
      "ricardian_contract": ""
    },{
      "name": "claim",
      "type": "claim",
      "ricardian_contract": ""
    },{
      "name": "setemergency",
      "type": "setemergency",
      "ricardian_contract": ""
    },{
       "name": "newaccount",
       "type": "newaccount",
       "ricardian_contract": ""
     },{
       "name": "setcode",
       "type": "setcode",
       "ricardian_contract": ""
     },{
       "name": "setabi",
       "type": "setabi",
       "ricardian_contract": ""
     },{
       "name": "updateauth",
       "type": "updateauth",
       "ricardian_contract": ""
     },{
       "name": "deleteauth",
       "type": "deleteauth",
       "ricardian_contract": ""
     },{
       "name": "linkauth",
       "type": "linkauth",
       "ricardian_contract": ""
     },{
       "name": "unlinkauth",
       "type": "unlinkauth",
       "ricardian_contract": ""
     },{
       "name": "canceldelay",
       "type": "canceldelay",
       "ricardian_contract": ""
     },{
       "name": "setconfig",
       "type": "setconfig",
       "ricardian_contract": ""
     },{
       "name": "onfee",
       "type": "onfee",
       "ricardian_contract": ""
     },{
       "name": "heartbeat",
       "type": "heartbeat_info",
       "ricardian_contract": ""
     },{
       "name": "removebp",
       "type": "removebp",
       "ricardian_contract": ""
     }
  ],
  "tables": [
        {
          "name":"accounts",
          "type":"account_info",
          "index_type": "i64",
          "key_names" : ["name"],
          "key_types" : ["account_name"]
       },{
          "name":"bps",
          "type":"bp_info",
          "index_type":"i64",
          "key_names": ["name"],
          "key_types": ["account_name"]
       },{
          "name":"votes",
          "type":"vote_info",
          "index_type":"i64",
          "key_names": ["bpname"],
          "key_types": ["account_name"]
       },{
          "name":"votes4ram",
          "type":"vote_info",
          "index_type":"i64",
          "key_names": ["bpname"],
          "key_types": ["account_name"]
       },{
          "name":"chainstatus",
          "type":"chain_status",
          "index_type":"i64",
          "key_names": ["name"],
          "key_types": ["account_name"]
       },{
          "name":"schedules",
          "type":"schedule_info",
          "index_type":"i64",
          "key_names": ["version"],
          "key_types": ["uint64"]
       },{
          "name":"heartbeat",
          "type":"heartbeat_info",
          "index_type":"i64",
          "key_names": ["bpname"],
          "key_types": ["account_name"]
       }
  ],
  "ricardian_clauses": []
}
```