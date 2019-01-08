# Cleos voteproducer说明

voteproducer是cleos的一个基本命令，用于给自己心仪的bp投票


## 测试链

- IP: 127.0.0.1

- Port：8888

使用示例: 

```bash
cleos -u http://127.0.0.1:8888 get info
```

## cleos voteproducer 命令参数

cleos voteproducer [选项] 子命令
```
选项:
  -h,--help                   打印帮助信息

Subcommands:
  vote                        投票
  list                        列出已投票的信息
  claim                       领取投票的分红
  unfreeze                    解冻已经赎回的投票
```
## cleos voteproducer 子命令详解
### vote
投票给bp节点
cleos voteproducer vote [选项] voter bpname amount
  voter				投票的账户
  bpname			接受投票的bp名称
  amount			投票的数量
使用示例: 
eosforce向biosbpa投票100
```bash
cleos voteproducer vote eosforce biosbpa 100
```
注意：
>这个功能的amount是给该bp最终的投票结果，也就是说如果之前投了10000票，执行示例命令则投票会变成100
### list
列出所有投票的信息
cleos voteproducer list [选项] voter
  voter 			投票的账户
使用示例: 
列出eosforce的所有投票信息
```bash
cleos voteproducer list eosforce
```
### claim 
领取在某个bp上投票的分红
cleos voteproducer claim [选项] voter bpname
  voter				投票的账户
  bpname		   	bp的名称
使用示例: 
eosforce领取在biosbpa上面的分红
```bash
cleos voteproducer claim eosforce biosbpa
```
注意：
>eosforce链上的操作是有手续费的，建议分红达到一定数量后再领取
### unfreeze
解冻赎回的投票
cleos voteproducer unfreeze [选项] voter bpname
  voter				投票的账户
  bpname		   	bp的名称
使用示例: 
eosforce解冻在biosbpa上面赎回的投票
```bash
cleos voteproducer unfreeze eosforce biosbpa
```
注意
>eosforce上的投票赎回后会有3天的冻结期，3天之后才能解冻
