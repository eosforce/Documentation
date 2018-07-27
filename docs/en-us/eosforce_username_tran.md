# 账户权限修改

## 功能概述

- 修改公钥（可用于实现用户名转让）
- 设置多个公钥（可用于实现多重签名）
- 修改公钥的权重
- 修改账户权限阈值

## 源码修改

- 手续费管理器增加一个action：updateauth, 并设置其手续费。
- 引入bios系统合约，controller类需增加初始化方法，以支持updateauth操作。

## RPC接口执行方法

调用 /v1/chain/push_transaction接口，提交updateauth action，修改账户owner权限、active权限。

> 账户默认会创建owner、active权限，也可以通过updateauth action自定义权限

## 命令行执行用例

1. 修改user1用户的公钥

/cleos set account permission -j user1 owner EOS6hj8ozvKetcfEPonMLdUm9Ey3HYPgc6Tt94R88BejE9ojbrzD5 -p ssh3@owner

2. 设置user1用户多重签名 (权限阈值为2，3个公钥)

/cleos set account permission -j user1 owner authority.json -p ssh3@owner

权限设置配置文件 authority.json：

```JSON
{
  "threshold": 2,
  "keys": [
    {
      "key": "EOS6BUSXxqmBBMxnCwFowfFsr8Zi1WRtWXguGzUb9oGGpueMSaJbx",
      "weight": 1
    },
    {
      "key": "EOS6hj8ozvKetcfEPonMLdUm9Ey3HYPgc6Tt94R88BejE9ojbrzD5",
      "weight": 1
    },
    {
      "key": "EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV",
      "weight": 1
    }
  ]
}
```

> 公钥(key)必须按字符升序无重复排列。eos代码中为方便统计公钥个数而限制。
>
> 所有公钥权重(weight)的和必须大于等于权限阈值(threshold)。小于阈值无法执行。