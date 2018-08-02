# newaccount 创建账号

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