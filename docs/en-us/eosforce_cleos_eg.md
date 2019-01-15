# common cleos commands

### 1. Get account balance:

```shell
./cleos get table eosio eosio accounts -k 账户名
```

> read from system contract accounts table

Returns:
```json
{
  "rows": [{
      "name": "b1",
      "available": "103241.0000 EOS"
    }
  ],
  "more": false
}
```

### 2. Get all corresponding accounts by public key 

```shell
./cleos get accounts public key
```

Returns:
```json
{
  "account_names": [
    "alice",
    "bob"
  ]
}
```
### 3. Get BP schedule

```shell
 ./cleos get table eosio eosio schedules -k version
```
```json
{
  "rows": [{
      "version": 2,
      "block_height": 4164,
      "producers": [{
          "bpname": "biosbpc",
          "amount": 0
        },{
          "bpname": "biosbpd",
          "amount": 0
        },{
          "bpname": "biosbpe",
          "amount": 6210
        }
      ]
    }
  ],
  "more": false
}
```
### 4. Get bp info
```shell
./cleos get table eosio eosio bps
./cleos get table eosio eosio bps -k node name
```
```json
{
  "rows": [{
      "name": "biosbpi",
      "block_signing_key": "EOS5sWyyriwYhg4iA6K8FxyiGwaCBYoUy2MAQHLEwo4AcdR31hXv9",
      "commission_rate": 1000,
      "total_staked": 10001,
      "rewards_pool": "52440.9600 EOS",
      "total_voteage": 258030000,
      "voteage_update_height": 25887,
      "url": "http://eosforce.io",
      "emergency": 0
    }
  ],
  "more": false
}
```

### 5. Get voting info
```shell
./cleos table eosio account votes
```
```json
{
  "rows": [{
      "bpname": "biosbpa",
      "staked": "33.0000 EOS",
      "voteage": 0,
      "voteage_update_height": 14238,
      "unstaking": "0.0000 EOS",
      "unstake_height": 14238
    },{
      "bpname": "biosbpi",
      "staked": "1.0000 EOS",
      "voteage": 0,
      "voteage_update_height": 14258,
      "unstaking": "0.0000 EOS",
      "unstake_height": 14258
    }
  ],
  "more": false
}
```

### 6. transfer

```shell
./cleos push action eosio transfer '{"from":"alice","to":"bob", "quantity":"1000.0000 EOS", "memo":"hello"}' -p alice@active
```

> submit System contract transfer transaction

### 7. Vote

```shell
./cleos push action eosio vote '{"voter":"alice","bpname":"bob","stake":"10.0000 EOS"}' -p alice@active
```
> submit System contract vote transaction，stake is the current number of votes cast for BP

### 8. Claim rewards

```shell
./cleos push action eosio claim '{"voter":"alice","bpname":"bob"}'  -p alice@active
```

> submit System contract claim transaction

### 9. unfreeze

```shell
./cleos push action eosio unfreeze '{"voter":"alice","bpname":"bob"}'  -p alice@active
```

> submit System contract unfreeze transaction 


> ./cleos push action encapsulates transaction request with an action。eosio is creating accounts for System system contracts, -p specify the execution account and permission for the current transaction。