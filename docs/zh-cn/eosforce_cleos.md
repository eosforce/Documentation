# Cleos说明

cleos是一个eos的命令行工具，封装了对nodeos的REST接口的调用，更方便使用。

cleos可通过编译eosforce项目后，在./build/program/cleos目录下得到。

使用cleos需要连接到一个nodeos的节点(IP地址:端口)，默认是本地8888端口。


## 测试链

- IP: 127.0.0.1

- Port：8888

使用示例: 

```bash
cleos -u http://127.0.0.1:8888 get info
```
> 测试多签需使用此代码编译的cleos： https://github.com/eosforce/eosforce/tree/feature/updateauth-msig

## cleos命令参数

cleos [选项] 子命令
```
选项:
  -h,--help                   打印帮助信息
  -u,--url TEXT=http://localhost:8888/
                              运行的nodeos的http/https URL
  --wallet-url TEXT=http://localhost:8900/
                              钱包管理器的http/https URL
  -r,--header                 http头
  -n,--no-verify              当使用https时不验证节点权限
  -v,--verbose                错误信息可视输出

Subcommands:
  version                     版本信心
  create                      创建变量、链开关
  [get](zh-cn/eosforce_cleos_get.md)                         获取信息
  [set](zh-cn/eosforce_cleos_set.md)                         设置修改链状态
  transfer                    转账命令
  [wallet](zh-cn/eosforce_cleos_wallet.md)                      管理本地钱包
  sign                        交易签名
  push                        提交一个二进制交易到链上
  [multisig](zh-cn/contract/eosio.msig/msig.md)                   多重签名，需使用eosforce最新代码编译cleos才能使用
  [system](zh-cn/eosforce_cleos_system.md)                        多用于更新bp以及和bp相关的一些操作
  [voteproducer](zh-cn/eosforce_cleos_voteproducer.md)		  给一个生产者投票
```
## cleos部分命令详解
### version
版本信息
cleos version SUBCOMMAND
client			检索客户端的版本信息
使用示例: 
```bash
cleos version client
```
### transfer
转账，将Token从一个账户转个另一个账户
cleos transfer [选项] sender recipient amount [memo]
sender			转出Token的账户
recipient		接收Token的账户
amount			转出的Token以及数量
memo			转账的备注
使用示例: 
eosforce给biosbpa转100个EOS
```bash
cleos transfer eosforce biosbpa "100.0000 EOS"
```
### create
创建账户/私钥
cleos create SUBCOMMAND
key		创建一个私钥
account		创建一个账户
#### create key
cleos create key [OPTIONS]
options:
-f,--file		创建的私钥和公钥输出的文件的名字
--to-console		在shell上打印出公钥和私钥
使用示例
创建一个新的私钥和公钥并打印在shell上面
```bash
cleos create key --to-console
```
#### create account
创建一个用户
cleos create account [OPTIONS] creator name OwnerKey [ActiveKey]
creator			创建账户使用的账户
name			新账户的名称
OwnerKey		新账户的OwnerKey
ActiveKey		新账户的ActiveKey
使用示例
使用eosforce创建一个名称为wang的账户
```bash
cleos create account eosforce wang EOS7pVjNGrg2MCdsULWEgMKbUDQfN8VLabTprEsDq4VTgrZjf7TGW
```
### push
push用来向链上提交一些操作
cleos push [OPTIONS] SUBCOMMAND
action				提交一个操作
transaction			提交一个事务
transactions			提交一系列事务
#### action
提交一个操作是使用最多的
cleos push action [OPTIONS] account action data
account				合约账户
action				操作的名称
data				操作所需要的数据
使用示例
通过push action完成eosforce向biosbpa的转账
```bash
cleos push action eosio transfer '["eosforce","biosbpa","100.0000 EOS",""]' -p eosforce@active
```
#### transaction
提交一个transaction
cleos push transaction [OPTIONS] transaction
transaction			用于提交的transaction  json格式的字符串或存放transaction的文件名
使用示例
通过push transaction完成eosforce向biosbpa的转账
```bash
cleos push transaction ./1.txt
```
```1.txt
 {
    "actions": [
        {
            "account": "eosio",
            "name": "transfer",
            "authorization": [
                {
                    "actor": "eosforce",
                    "permission": "active"
                }
            ],
            "data": {
                "from": "eosforce",
                "to": "biosbpa",
                "quantity": "10.0000 EOS",
                "memo": "hello"
            }
        }
    ],
    "transaction_extensions":[]
}
```
### sign
给transaction 签名
cleos sign [OPTIONS] transaction
transaction			用于签名的transaction  json格式的字符串或存放transaction的文件名
使用示例
通过push action完成eosforce向biosbpa的转账
```bash
./cleos sign ./trx.txt  -k 5KhfDXX2iXR54ktXVuuCC7a4UJxAZaSxnH4BnEfuMRbBMH576TW
```
### get
详情参见[get](zh-cn/eosforce_cleos_get.md) 
### set
详情参见[set](zh-cn/eosforce_cleos_set.md)
### wallet
详情参见[wallet](zh-cn/eosforce_cleos_wallet.md)
### multisig
详情参见[multisig](zh-cn/contract/eosio.msig/msig.md)
### system
详情参见[system](zh-cn/eosforce_cleos_system.md)
### voteproducer
详情参见[voteproducer](zh-cn/eosforce_cleos_voteproducer.md)



