# Cleos set说明

set是cleos的一个基本命令，用于设置一些基本信息


## 测试链

- IP: 127.0.0.1

- Port：8888

使用示例: 

```bash
cleos -u http://127.0.0.1:8888 get info
```

## cleos set 命令参数

cleos set [选项] 子命令
```
选项:
  -h,--help                   打印帮助信息

Subcommands:
  code                   设置或更新账户的code信息
  abi                    设置或更新账户的abi信息
  setfee                 设置或更新一个操作的费用
  contract               给账户设置衣服合约
  account		 设置一个账户的权限
  action		 给action设置权限
```
## cleos set 子命令详解
### code
设置用户的code信息
cleos set code [选项] account [code-file]
  account			账户的名称
  code-file			code文件的路径
使用示例: 
给biosbpa设置hello合约
```bash
cleos set code biosbpa ../../contracts/hello/hello.wasm
```
### abi
设置用户的abi信息
cleos set abi [选项] account [abi-file]
  account			账户的名称
  abi-file			code文件的路径
使用示例: 
给biosbpa设置hello合约
```bash
cleos set abi biosbpa ../../contracts/hello/hello.abi
```
### setfee 
设置或更新一个操作的费用
cleos set setfee [选项] account action fee cpu_limit net_limit ram_limit
  account 			合约账户的名称
  action			action的名称
  fee				费用
  cpu_limit			cpu的使用限制
  net_limit			net的使用限制
  ram_limit			ram的使用限制
使用示例: 
给biosbpa的hello操作设置费用
```bash
cleos set setfee biosbpa hello "0.0010 EOS" 1000000 1000000 1000
```
注意：
>setfee需要的权限是eosio，所以一般只有官方或者所有bp的多签才能使用该功能
### contract
设置合约信息  内部调用的是setcode和setabi
cleos set contract [选项] account [contract-dir] [wasm-file] [abi-file]
  account 			合约账户的名称
  contract-dir			合约文件所在的路径
  wasm-file			wasm文件所在的路径
  abi-file			abi文件所在的路径
使用示例: 
```bash
cleos set contract biosbpa ../../contracts/hello
```
### account
设置一个账户的权限
cleos set account permission [选项] account permission [authority] [parent]
account			需要设置权限的账户
permission		新的权限的信息
authority		公钥，json,文件名用来定义新的权限
parent			该权限的父权限
使用示例:
将biosbpb的active权限修改为biosbpa和biosbpc的active权限
```bash
cleos set account permission biosbpb active '{ "threshold": 2, "keys": [], "accounts":[ { "permission": { "actor": "biosbpa", "permission": "active" }, "weight": 1 }, { "permission": { "actor": "biosbpc", "permission": "active" }, "weight": 1 } ] }' owner
```
### action
给合约的一个action设置权限
cleos set action permission [OPTIONS] account code type requirement
account			合约的账户
code			设置的权限的账户的名称
type			合约的action的名字
requirement		权限的名字
使用示例:
将biosbpa的hi操作的权限设置为biosbpb的active
```bash
cleos set action permission biosbpa biosbpb hi active
```




