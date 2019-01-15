# account permission modification

## Function Overview

- Modify public key（can be used for account transfer）
- Setting up multiple public keys or granting accounts（can be used for multisig）
- Modify public key's weight
- Modify accout permission threshold

## RPC interface

Calling /v1/chain/push_transaction interface，submit updateauth action，modify account owner permssion、active permission。

abi data structure：

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
permission data structure：
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
- threshold: permission threshold
- keys: public key weight array
- accounts: account weight array
- waits: wait time weight array

account weight
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
public key weight
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
> Accounts create owner, activity permissions by default, can also customize permissions by updateauth action


## modify user permission using cleos command
```bash
cleos set account permission [OPTIONS] account permission authority [parent]

Options:
  -h,--help                   Print this help message and exit

  -x,--expiration             set the time in seconds before a transaction expires, defaults to 30s

  -f,--force-unique           force the transaction to be unique. this will consume extra bandwidth and remove any protections against accidently issuing the same transaction multiple times

  -s,--skip-sign              Specify if unlocked wallet keys should be used to sign transaction

  -j,--json                   print result as json 

  -d,--dont-broadcast         don't broadcast transaction to the network (just print to stdout)

  -r,--ref-block TEXT         set the reference block num or block id used for TAPOS (Transaction as Proof-of-Stake)

  -p,--permission TEXT ...    An account and permission level to authorize, as in 'account@permission' (defaults to 'account@active')
  
Parameters
  account TEXT                The account to set/delete a permission authority for (required)

  permission TEXT             The permission name to set/delete an authority for (required)

  authority TEXT              [delete] NULL, [create/update] public key, JSON string, or filename defining the authority (required)

  parent TEXT                 [create] The permission name of this parents permission (Defaults to: "Active")
```

## Example

1. modify user1's public_key

/cleos set account permission -j user1 owner EOS6hj8ozvKetcfEPonMLdUm9Ey3HYPgc6Tt94R88BejE9ojbrzD5 -p user1@owner

2. set user1's multple signature (permission threshold: 2, 3 public_keys)

/cleos set account permission -j user1 owner authority.json -p user1@owner

permission Setting configuration file authority.json：

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

> Public key must be arranged in ascending order uniquely. Limitation in EOS code for the convenience of counting the number of public keys。
>
> Sum of total public key weight must be greater than or equal to permission threshold。