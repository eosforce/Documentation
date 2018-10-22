# 合约开发

## 合约编译上传流程

```shell
account='你的合约账号'
eosiocpp=./build/tools/eosiocpp #eosiocpp工具路径，可由eosforce项目编译后产生
contract_path='你的合约源码路径'

# 编译
$eosiocpp -g contract.abi $contract_path/contract.hpp
$eosiocpp -o contract.wasm $contract_path/contract.cpp

# 上传
leos wallet unlock --password '钱包密码'
cleos -u https://w1.eosforce.cn set abi $account contract.abi
cleos -u https://w1.eosforce.cn set code $account contract.wasm
```
