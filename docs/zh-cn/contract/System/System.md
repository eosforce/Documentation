# eosforce系统合约

> 替换原eosio.system系统合约

## action 合约方法

### 转账
```C++
void transfer( const account_name from, const account_name to, const asset quantity, const string memo );
```
### 设置BP（区块生产者）
```C++
void updatebp( const account_name bpname, const public_key producer_key,
const uint32_t commission_rate, const std::string & url );
```

### 投票
```C++
void vote( const account_name voter, const account_name bpname, const asset stake );
```
1. 设置用户的投票信息：
投票数大于当前票数，增加投票；
投票数小于当前票数，撤回投票；
2. 根据投票数减少相应用户余额
3. 增加节点的总票数， 结算节点当前总票龄

### 解冻，
```C++
void unfreeze( const account_name voter, const account_name bpname );
```
    撤回投票要冻结3天(根据块数确定，每3秒一块)
    将撤回票数 加到可用余额中。清空撤回票数。


### 领取分红
```C++
void claim( const account_name voter, const account_name bpname );
```
1. 计算投票账号最新票龄：上次票龄+票数*(当前块高度-上次结算时块高度)
2. 计算节点的总票龄：上次票龄+票数*(当前块高度-节点上次结算时块高度)
3. 计算分红数：节点奖池总数*投票者总票龄/节点总票龄
4. 增加投票账号余额 +=分红数
5. 投票信息票龄清零，更新结算块高度
6. 减少节点奖池 -=分红数，总票龄减少账号票龄，更新计算块高度


### 出块，每次出块时回调    (系统内部使用)
```C++
void onblock(…const account_name bpname,const uint32_t schedule_version);
```
1. 出块节点bpname，producer.amount出块数加1
2. 每个出块奖励9个EOS，按分红比例，将奖励加入出块节点余额，并将剩余分红加入出块节点的奖池中。
3. 每个出块将奖励1个EOS给b1账号(block.one)
4. 刷新选举的23个超级节点，遍历所有bp查前23个，调用wasm接口set_proposed_producers重置BP,修改数据库数据

### 扣除手续费    (系统内部使用)
```C++
void onfee( const account_name actor, const asset fee, const account_name bpname );
```
所有操作的手续费最终调此action执行，根据手续费扣除账号余额，并增加到打包此区块的节点余额中。

### 设置链紧急状态
```C++
void setemergency( const account_name bpname, const bool emergency );
```
超过三分之二的节点设置紧急状态，则链将处于紧急状态。将紧急停用所有功能。

## 表
```json
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
       }
  ]
```

## 数据结构
```json
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
      "name": "vote_parameter",
      "base": "",
      "fields": [
        {"name":"voter",     "type":"account_name"},
        {"name":"bpname",    "type":"account_name"},
        {"name":"stake",     "type":"asset"}
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
    }
  ]
```