# account transfer

## 1 off-chain transaction

Modify account's public key using [updateauth](en-us/contract/eosio.bios/updateauth.md)，public key is provided by others.

> Note：eosforce transaction can only include one action，so account transfer need to submit two transactions in order。

one side：first modify active permission，then modify onwer permission. After modifying owner permission, no modifications can be executed any more。
two sides：The owner modifies the owner permission to the recipient's public key; the recipient modifies the active permission of this account



## 2 on-chain transaction

(not implmented yet)
account transfer smart contract nametransfer
action：
1. The account owner submits an application for transfer and sets up the account name, transfer fee and designated purchaser account to be transferred.
(optional)。
void sell_name(account, price, buyer);
2. Purchaser executes purchase transfer
void buy_name(account);


## 3 account auction

(not implmented yet)
account auction smart contract
1. Registration slip (account name, base price, each price increase, duration)
2. Bidding (account name, bid)
3. Bidding Status Query (Account Name)
4. Cancel Registration Form (Account Name)


> The short-name account (less than 12 characters) bidding function provided by eosio is only the creation of short-name bidding. Uncreated short names must be authorized to create this account by bidding first.