# Multiple Signature

## Function Overview

eos sccount and permission system support multiple signature function，An account has owner、active permissions by default when created，every pemission 's  threshold is 1，is  comprise of public key，every key's weight is 1。when the key weight's sum are great than or equal to permission threshold，the permission can be granted successfully。Default's configuration requires one signature to be granted to execute some actions, because every key's weight is great than or equal to threshold。

```bash
./cleos get account user1
permissions:
     owner     1:    1 EOS8T5BG84e9L9BdUFxg7a5xiTfbE51zBPt4mKkesuhm4ZWx7jAe5
        active     1:    1 EOS8T5BG84e9L9BdUFxg7a5xiTfbE51zBPt4mKkesuhm4ZWx7jAe5
```

Suppose account permission has my keys, and permission threshold is greater than every key's single weight, permission could be granted only when multiple keys are signed.

For example：

Setting up ssh3 account active permission threshold to 2，accounts to alice and bob's active permission，weight is 1。Thus actions that can be executed by ssh3 account can only be granted by alice and bob together.

```bash
./cleos set account permission ssh3 active '{ "threshold": 2, "keys": [], "accounts":[ { "permission": { "actor": "alice", "permission": "active" }, "weight": 1 }, { "permission": { "actor": "bob", "permission": "active" }, "weight": 1 } ] }' owner

./cleos get account ssh3
permissions:
     owner     1:    1 EOS8T5BG84e9L9BdUFxg7a5xiTfbE51zBPt4mKkesuhm4ZWx7jAe5
        active     2:    1 alice@active, 1 bob@active,
```

> account permissions can be set to public keys，also can be other accounts.。

## eosio.msig contract

When  multisig account's key is distributed in different wallets，an asynchronous collaboration mechanism is need to execute the transaction. EOS provides eosio.msig contract，the contract provides synchronous-side channel，data was signed and transfered in this channel。 Eosio.msig proposes an proposal asynchronous、approve and execute the transaction finally。

**eosio.msig** contract action：

1. propose
2. approve
3. unapprove
4. exec
5. cancel

## multiple signature execution process

### 1. initiate a proposal

```bash
./cleos multisig propose pab1 '[{"actor":"alice","permission":"active"},{"actor":"bob","permission":"active"}]' '[{"actor":"ssh3","permission":"active"}]' eosio transfer '{"from":"ssh3","to":"ssh","quantity":"66.0000 EOS","memo":"msig transfer"}' ssh2
```

cleos multisig propose parameters：

- proposal_name 
- requested_permissions, json format
- trx_permissions, json format
- contract: contract to which deferred transaction should be delivered
- action: action of deferred transaction
- data: The JSON string or filename defining the action to propose
- proposer:  Account proposing the transaction

### 2. approve the proposal

```bash
./cleos multisig approve ssh2 pab1 '{"actor":"alice","permission":"active"}' -p alice@active
./cleos multisig approve ssh2 pab1 '{"actor":"bob","permission":"active"}' -p bob@active
```

cleos multisig approve parameters：

- proposer:  proposer name
- proposal_name: proposal name
- permissions: The JSON string of filename defining approving permissions

### 3. execute the proposal

```bash
./cleos multisig exec ssh2 pab1 ssh2
```

cleos multisig exec parameters：

- proposer: proposer name
- proposal_name: proposal name
- executer: account paying for execution

Note：cleos can be used when compiled by Eosforce recent source，old cleos can not be used because msig command has not added fee.

## example script:

```bash

#!/bin/bash

##########################################################################
# This is a EOS multi sign demo script.
#
# eosforce.io
#
##########################################################################

echo '---------- this is a msig demo'
echo '---------- unlock wallet'
./cleos wallet unlock

echo '---------- create account : alice and bob, and transfer 0.3 EOS'
./cleos create account ssh alice EOS5x4UDqvUsjWXySFqzboR7GYmwipkXV4sgGpgDRqouzd7NprQ5m
./cleos create account ssh bob EOS6BUSXxqmBBMxnCwFowfFsr8Zi1WRtWXguGzUb9oGGpueMSaJbx
./cleos push action eosio transfer '{"from":"ssh","to":"alice", "quantity":"0.3000 EOS", "memo":""}' -p ssh@active
./cleos push action eosio transfer '{"from":"ssh","to":"bob", "quantity":"0.3000 EOS", "memo":""}' -p ssh@active

echo '---------- set ssh3 msig permisson'
#./cleos set account permission ssh3 active d/my_authority_acc.json owner
./cleos set account permission ssh3 active '{ "threshold": 2, "keys": [], "accounts":[ { "permission": { "actor": "alice", "permission": "active" }, "weight": 1 }, { "permission": { "actor": "bob", "permission": "active" }, "weight": 1 } ] }' owner
./cleos get account ssh3

echo '---------- msig propose : ssh3 transfer ssh 60 EOS (by ssh2)'
./cleos multisig propose -j pab1 '[{"actor":"alice","permission":"active"},{"actor":"bob","permission":"active"}]' '[{"actor":"ssh3","permission":"active"}]' eosio transfer '{"from":"ssh3","to":"ssh","quantity":"66.0000 EOS","memo":"msig transfer"}' ssh2

echo '---------- msig approve: alice and bob'
./cleos multisig approve ssh2 pab1 '{"actor":"alice","permission":"active"}' -p alice@active
./cleos multisig approve ssh2 pab1 '{"actor":"bob","permission":"active"}' -p bob@active

echo '---------- get account balance before msig exe'
./cleos get table eosio eosio accounts -k ssh
./cleos get table eosio eosio accounts -k ssh3

echo '---------- msig exe'
./cleos multisig exec ssh2 pab1 ssh2

echo 'sleep 3 ...'
sleep 1
echo 'sleep 2 ..'
sleep 1
echo 'sleep 1 .'
sleep 1

echo '---------- get result'
./cleos get table eosio eosio accounts -k ssh
./cleos get table eosio eosio accounts -k ssh3

echo '---------- maybe not pack in the block yet, try again later (./cleos get table eosio eosio accounts -k ssh)'


```