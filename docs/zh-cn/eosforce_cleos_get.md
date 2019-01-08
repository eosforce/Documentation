# Cleos get说明

get是cleos的一个基本命令，用于查询链上的一些信息

## 测试链

- IP: 127.0.0.1

- Port：8888

使用示例: 

```bash
cleos -u http://127.0.0.1:8888 get info
```

## cleos get 命令参数

cleos get [选项] 子命令
```
选项:
  -h,--help                   打印帮助信息

Subcommands:
  info                        获取链的基本信息
  block                       从链上获取某一个区块的信息
  account                     获取某一个账户的信息
  code                        从一个合约账户获取code
  abi			      从一个合约账户获取abi信息
  table			      获取一个表的内容
  scope			      检索合约拥有的范围和表的列表
  accounts		      根据public_key查看所有使用该public_key的账户
  transaction		      获取一个transaction的信息
  actions		      获取一个账户的所有动作的信息
  schedule		      获取当前出块节点的信息
```
## cleos get 子命令详解
### info
获取链的基本信息
cleos get info
使用示例: 
```bash
cleos get info
```
### block
获取一个块的信息
cleos get block [选项] block
  block 			想要查询的block的编号
使用示例: 
```bash
cleos get block 2
```
### account 
获取一个账户的信息
cleos get account [选项] name
  name				要查询的账户的名字
使用示例: 
查询eosforce的信息
```bash
cleos get account eosforce
```
### code
从一个合约账户获取code
cleos get code [选项] name
  name				获取code的合约账户
使用示例: 
获取eosio.token的code
```bash
cleos get code eosio.token
```
注意
>此处获取的是code的Hash
### abi 
获取一个合约账户的abi
cleos get abi [选项] name
  name				获取abi的合约账户
使用示例: 
获取eosio.token的abi
```bash
cleos get abi eosio.token
```
### table 
获取一个table中的内容
cleos get table [选项] account scope table
  account			拥有表格的合约账户
  scope				查询表格的范围
  table				查询的表格的名字
使用示例: 
查询eosio上面的表格accounts中的内容
```bash
cleos get table eosio eosio accounts
```
### scope
检索合约拥有的范围和表的列表
cleos get scope [选项] contract
contract		拥有表格的合约账户信息
使用示例: 
查询eosio拥有的表以及范围
```bash
cleos get scope eosio
```
### accounts
根据public_key查看所有使用该public_key的账户
cleos get accounts public_key
public_key		公钥
使用示例: 
```bash
cleos get accounts EOS6k7Es5WqmntPf8qHtpDaNYYsaadrvfxjn8jGKa8tv8voSqazGu
```
### transaction
根据transaction_id获取一个transaction的信息
cleos get transaction [OPTIONS] id
id		transaction_id
使用示例: 
```bash
cleos get transaction 42fe264a09f280002746939c3083e98d6362eac0b5339c7b779bdbadc907c286
```
### actions
获取一个账户的actions
cleos get actions [OPTIONS] account_name [pos] [offset]
account_name			获取action的账户
pos				获取action的位置
offset				获取action的偏移位置
使用示例: 
```bash
cleos get actions eosforce 1 2
```
### schedule
获取当前出块节点的信息
cleos get schedule [OPTIONS]
使用示例: 
```bash
cleos get schedule 
```

