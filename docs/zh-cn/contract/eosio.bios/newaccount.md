# newaccount 创建账号

## 创建规则

账户名不超过12位，并且只能由这些字符组成：.12345abcdefghijklmnopqrstuvwxyz

> 注意：提交了创建用户名的交易，需等待到进入不可逆块。如果有其他人抢注这个账号，这个账号可能不属于你，如果刚创建就马上就给这个账号转账的话，可能钱就打水漂啦。

## cleos 命令

```bash
cleos create account [OPTIONS] creator name OwnerKey [ActiveKey]
```

Positionals:
  - creator TEXT                The name of the account creating the new account (required)
  - name TEXT                   The name of the new account (required)
  - OwnerKey TEXT               The owner public key for the new account (required)
  - ActiveKey TEXT              The active public key for the new account

参数：
  - creator TEXT                执行创建的账号 (必要)
  - name TEXT                   创建的新账号名 (required)
  - OwnerKey TEXT               拥有者的账号公钥 (required)
  - ActiveKey TEXT              新账号的激活公钥，默认同拥有者的账号公钥


## abi 数据结构
- [eosio.bio.abi](https://github.com/eosforce/eosforce/blob/release/contracts/eosio.bios/eosio.bios.abi)
  
```json
    {
      "name": "newaccount",
      "base": "",
      "fields": [
        {"name":"creator", "type":"account_name"},
        {"name":"name",    "type":"account_name"},
        {"name":"owner",   "type":"authority"},
        {"name":"active",  "type":"authority"}
      ]
    }

```