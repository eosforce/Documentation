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

- vote_info：设置用户的投票信息：
增加{节点,票数}> or 
修改{票龄,票数}:追加\撤回{+撤回票数}
- account_info：减少相应用户余额
- bp_info：节点的总票数， 结算当前总票龄


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


### 出块，每此出块时回调，仅系统调用
```C++
void onblock(…const account_name bpname,const uint32_t schedule_version);
```
1. 出块节点bpname，producer.amount出块数加1
2. 每个出块奖励9个EOS，按分红比例，将奖励加入出块节点余额，并将剩余分红加入出块节点的奖池中。
3. 每个出块将奖励1个EOS给b1账号(block.one)
4. 刷新选举的23个超级节点，遍历所有bp查前23个，调用wasm接口set_proposed_producers重置BP,修改数据库数据，这个是系统用的

### 收手续费
```C++
void onfee( const account_name actor, const asset fee, const account_name bpname );
```

### 设置链紧急状态
```C++
void setemergency( const account_name bpname, const bool emergency );
```
