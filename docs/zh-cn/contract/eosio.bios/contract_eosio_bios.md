# eosio.bios 系统合约

通过此合约eos系统可以直接控制其它账户的资源分配，并且可以调用其它需要特殊权限的API。
主要包括账号与权限设置，合约设置，资源分配等

#### 合约方法 actions

- [newaccount](zh-cn/contract/eosio.bios/newaccount.md)  : 创建账号 
- [updateauth](zh-cn/contract/eosio.bios/updateauth.md)  : 修改权限
- deleteauth : 删除权限

> 以下eosforce未开放

- setcode : 设置合约code
- setabi : 设置合约abi
- linkauth : 链接权限
- unlinkauth : 取消权限链接
- canceldelay : 
- onerror

- setalimits
- setglimits
- setpriv
- setprods
- reqauth



#### 实现源码

- [eosio/chain/eosio_contract.hpp](https://github.com/eosforce/eosforce/blob/master/libraries/chain/include/eosio/chain/eosio_contract.hpp)
- [eosio/chain/eosio_contract.cpp](https://github.com/eosforce/eosforce/blob/master/libraries/chain/eosio_contract.cpp)