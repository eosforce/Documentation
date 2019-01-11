# eosforce System 系统合约

## 合约说明
System合约是eosforce系统关键功能的实现合约，替换原eosio.system系统合约。
包括投票、分红等，对eos原有逻辑的修改。
改合约使用eosio账户发布，固定在初始化代码中。

## 合约方法 action

### 转账
```C++
void transfer( const account_name from, const account_name to, const asset quantity, const string memo );
```
参数：
- from : 转账账号
- to : 收款账号
- quantity : 金额，必须是 EOS资产
- memo : 备注, 必须小于256字节
  
### 设置BP（区块生产者）
```C++
void updatebp( const account_name bpname, const public_key producer_key,
const uint32_t commission_rate, const std::string & url );
```
  添加、修改节点信息

参数：
- bpnbame : 节点账号
- producer_key : 节点公钥
- commission_rate : 分红比列 单位:万分之1, 范围:大于等于1，小于等于10000
- url : 官网链接， 不超过64字节

### 投票
```C++
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

### 解冻
```C++
void unfreeze( const account_name voter, const account_name bpname );
```
    撤回投票要冻结3天(根据块数确定，每3秒一块)，3天后可执行解冻。

逻辑：

将撤回票数 加到可用余额中。清空撤回票数。


### 领取分红
```C++
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


### 出块回调函数    (系统内部使用)
```C++
void onblock(…const account_name bpname,const uint32_t schedule_version);
```
参数:
- bpname : 节点名
- schedule_version : 节点排行换届版本
  
逻辑:
1. 出块节点bpname，producer.amount出块数加1
2. 每个出块奖励9个EOS，按分红比例，将奖励加入出块节点余额，并将剩余分红加入出块节点的奖池中。
3. 每个出块将奖励1个EOS给b1账号(block.one)
4. 刷新选举的23个超级节点，遍历所有bp查前23个，调用wasm接口set_proposed_producers重置BP,修改数据库数据
 
### 设置链紧急状态
```C++
void setemergency( const account_name bpname, const bool emergency );
```

超过三分之二的节点设置紧急状态，则链将处于紧急状态。将紧急停用所有功能。
参数: 
- bpname : 节点
- emergency : 紧急状态
 
## abi 表
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

## abi 数据结构
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