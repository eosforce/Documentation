# eos账户与权限

eos是一个面向账号的区块链操作系统，账号与权限体系是eos系统最重要的支撑结构。
账户权限相关功能对外开放的操作都是通过 [eosio.bios](contract_eosio_bios.md) 这个特殊的系统合约在实现的。


#### eosio.bios 账号权限相关操作 actions

- [newaccount](newaccount.md) : 创建账号 
- [updateauth](updateauth.md) : 修改权限
- deleteauth : 删除权限

> 以下操作eosforce暂未开放

- setcode : 设置合约code
- setabi : 设置合约abi
- linkauth : 链接权限
- unlinkauth : 取消权限链接


####  源码

- [eosio/chain/eosio_contract.hpp](https://github.com/eosforce/eosforce/blob/master/libraries/chain/include/eosio/chain/eosio_contract.hpp)
- [eosio/chain/eosio_contract.cpp](https://github.com/eosforce/eosforce/blob/master/libraries/chain/eosio_contract.cpp)