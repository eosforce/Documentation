# Eosio.pledge description

Eosio.pledge is a contract for recording mortgage information that allows other contracts to call and query the mortgage status of the relevant account.

## Features provided by eosio.pledge

- addtype       Add mortgage category
- addpledge     Increase mortgage
- deduction     Deducting deposit
- withdraw      Take out the deposit
- getreward     Receive award
- allotreward   Distribution reward
- open          Create a mortgage entry
- close         Delete empty mortgage entry

### addtype

Related parameters
- pledge_name           Mortgage category name
- deduction_account     Account that can be deducted from the deposit
- ram_payer             Memory payer
- quantity              Currency of the deposit
- memo                  Remarks

Example
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge addtype '["block.out","eosio","eosio","2.0000 EOS","system pledge"]' -p eosio
```

Note: Currently addtype only allows eosio accounts, other accounts do not have access

### addpledge

Related parameters
- from                  Account with deposit
- to                    Contract account
- quantity              Amount of deposit
- memo                  Mortgage category name

Example
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge open '["block.out","eosforce","testopen"]' -p eosforce
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 transfer eosforce eosio.pledge "20000.0000 EOS" "block.out"
```

Note：addpledge is notified by transfer action on eosio contract,if to account on transfer is eosio.pledge addpledge will be called，Normal user cannot call.If the first call needs to call open to add the mortgage entry, the mortgage amount can be updated to the mortgage entry.

### deduction

Related parameters
- pledge_name           Mortgage category name
- debitee               Account name who was deducted from the deposit
- quantity              Amount deducted from the deposit
- memo                  Remarks

Example
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge deduction '["block.out","biosbpa","1.0000 EOS","test deduction"]' -p eosio
```

Note: Only the deduction_account account has the right to deduct the deposit.

### withdraw

- pledge_name           Mortgage category name
- pledger               Account for deposit
- quantity              The amount of the deposit
- memo                  Remarks

Example
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge withdraw '["block.out","eosforce","1.0000 EOS","test withdraw"]' -p eosforce
```

Note: The user can only receive the amount of his mortgage, and the part deducted by deduction_account cannot be collected.

### getreward

Related parameters
- rewarder              Rewarding account
- quantity              The currency in which the reward is received
- memo                  Remarks

Example
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge getreward '["eosforce","5.0000 EOS","test reward"]' -p eosforce
```

Note: All rewards will be collected for each collection. Quantity only represents currency

### allotreward

Related parameters
- pledge_name           Mortgage category name
- pledger               Mortgage account
- rewarder              Reward  account
- quantity              Amount of reward
- memo                  Remarks

Example
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge allotreward '["block.out","eosforce","eosforce","1.0000 EOS","test allotreward"]' -p eosio
```

Note: Only the deduction_account account has the ability to distribute rewards. The amount of bonuses distributed must not exceed the amount deducted by pledger.

### open

Related parameters
- pledge_name           Mortgage category name
- payer                 Account name who open a mortgage entry
- memo                  Remarks

```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge open '["block.out","eosforce","testopen"]' -p eosforce
```

Note: Each account only needs to be open once on a mortgage category, and multiple open will not report an error.

### close

相关参数
- pledge_name           Mortgage category name
- payer                 Account name who open a mortgage entry
- memo                  Remarks

```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge close '["block.out","eosforce","testopen"]' -p eosforce
```

Note: Only items that are unsecured and not debited can be closed.








