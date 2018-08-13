# 账户权限修改

## 功能概述

- 修改公钥（可用于实现用户名转让）
- 设置多个公钥或授权账号（可用于实现多重签名）
- 修改公钥的权重
- 修改账户权限阈值

## RPC接口执行方法

调用 /v1/chain/push_transaction接口，提交 updateauth action，修改账户owner权限、active权限。

```json
{
  "compression": "none",
  "transaction": {
    "expiration": "2018-08-09T10:21:21",
    "ref_block_num": 24885,
    "ref_block_prefix": 3198095408,
    "net_usage_words": 0,
    "max_cpu_usage_ms": 0,
    "delay_sec": 0,
    "context_free_actions": [],
    "actions": [
      {
        "account": "eosio",
        "name": "updateauth",
        "authorization": [
          {
            "actor": "yg",
            "permission": "owner"
          }
        ],
        "data": "00000000000000f300000000a8ed32320000000080ab26a70100000001000301cf1d9b6abc1b647c06446e7fceccaec6e5ac78592049751b95834de5dd6e7301000000"
      }
    ],
    "transaction_extensions": [],
    "fee": "0.1000 EOS"
  },
  "signatures": [
    "SIG_K1_KZtvqcfubxXciq6dQWgoVsdmo8TtF56yV39nDHpGV9bxiHWxSbugX2WyUFjgX436i5Ri8YkL478E6NaV66xZ5JuVgM3KcG"
  ]
}
```

abi数据结构：
- [eosio.bio.abi](https://github.com/eosforce/eosforce/blob/release/contracts/eosio.bios/eosio.bios.abi)

```json
{
      "name": "updateauth",
      "base": "",
      "fields": [
        {"name":"account",    "type":"account_name"},
        {"name":"permission", "type":"permission_name"},
        {"name":"parent",     "type":"permission_name"},
        {"name":"auth",       "type":"authority"}
      ]
    }
```
权限数据结构
```json
{
      "name": "authority",
      "base": "",
      "fields": [
        {"name":"threshold", "type":"uint32"},
        {"name":"keys",      "type":"key_weight[]"},
        {"name":"accounts",  "type":"permission_level_weight[]"},
        {"name":"waits",     "type":"wait_weight[]"}
      ]
    }
```
- threshold 权限阈值
- keys 公钥权重数组
- accounts 账号权重数组
- waits 等待时长权重数组

账号权重
```json
{
  "name": "permission_level_weight",
  "base": "",
  "fields": [
    {"name":"permission", "type":"permission_level"},
    {"name":"weight",     "type":"weight_type"}
  ]
},
{
  "name": "permission_level",
  "base": "",
  "fields": [
    {"name":"actor",      "type":"account_name"},
    {"name":"permission", "type":"permission_name"}
  ]
    }
```
公钥权重
```json
{
      "name": "key_weight",
      "base": "",
      "fields": [
        {"name":"key",    "type":"public_key"},
        {"name":"weight", "type":"weight_type"}
      ]
    }
```
> 账户默认会创建owner、active权限，也可以通过updateauth action自定义权限

## 修改用户权限cleos命令
```bash
cleos set account permission [OPTIONS] account permission authority [parent]

Options:
  -h,--help                   打印帮助信息

  -x,--expiration             transaction有效时间，单位：秒，默认30秒。

  -f,--force-unique           强制交易不重复，将消耗带宽并且移除多次执行同一个交易的意外情况的所有保护

  -s,--skip-sign              如果解锁钱包，需要签名交易指定key

  -j,--json                   json格式输出  

  -d,--dont-broadcast         不广播消息到网络

  -r,--ref-block TEXT         引用的使用区块号用于股份证明 TAPOS (Transaction as Proof-of-Stake)

  -p,--permission TEXT ...    认证的账号和权限 'account@permission' (默认 'account@active')
  
参数
  account TEXT                设置权限的账户 (必要)

  permission TEXT             设置的权限 (必要)

  authority TEXT              [删除] NULL, [创建或修改] 公钥key、 JSON字符串或文件名定义的权限(必要)

  parent TEXT                 [创建] 创建此权限的父权限(默认: "Active")
```

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