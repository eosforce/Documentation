# eosforce System Contract

## Contract explained
System Contract is the contract that eosforce implements the key system functions，replacing original eosio.system system contract.
including modifications to original EOS's original logic on voting、rewarding etc.
System contract was released using eosio account and was fixed in initializatin code.

## Contract action

### transfer
```C++
void transfer( const account_name from, const account_name to, const asset quantity, const string memo );
```
Parameters：
- from : originating account
- to : receipient account
- quantity : amount，must be EOS assets
- memo : memo, must less than 256 bytes
  
### Setting up BP（Block Producer）
```C++
void updatebp( const account_name bpname, const public_key producer_key,
const uint32_t commission_rate, const std::string & url );
```
  Add、modify node information

Parameters：
- bpnbame : node account
- producer_key : node public_key
- commission_rate : dividend ratio unit:one out of ten thousand, scope:great than or equal to 1，less than or equal to 10000
- url : website url， no more than 64 bytes

### Voting
```C++
void vote( const account_name voter, const account_name bpname, const asset stake );
```
  Voting for super node

Parameters:
- voter : voter
- bpbame : node
- stake : votes（eos amount）

logic:
1. setting up voting information：
votes greater than current votes，increase votes；
votes less than current votes，revoke votes；
2. decrease available balance according to votes
3. add node's total votes， computes node's current vote age

### 4. Change Vote

**Change Vote** Can allow user change their votes from one bp to another without freeze their token.

```cpp
void revote( const account_name voter, 
             const account_name frombp, 
             const account_name tobp, 
             const asset restake ) {
```

Parameters:

- voter : voter
- frombp : the bp voted
- tobp : the bp to vote
- restake : change votes（eos amount）

Example:

Suppose the user `testa` voted the `biosbpa` node with `5000.0000 EOS`,
At this point, the user wants to switch the votes to the `biosbpb` node with `2000.0000 EOS`, then user can execute:

```bash
cleos push action eosio revote '{"voter":"testa","frombp":"biosbpa","tobp":"biosbpb","restake":"2000.0000 EOS"}' -p testa
```

After execution, the user's Token for the `biosbpa` node is `3000.0000 EOS`, and the Token voted for the `biosbpb` node is increased by `2000.0000 EOS`.

### unfreeze
```C++
void unfreeze( const account_name voter, const account_name bpname );
```
    Revoking voting will freeze stake for 3 days(determined by block num，one block per 3 seconds)，stakes will be unfrozen 3 days later。

logic：

Revoked votes will be added to available balance。


### claim  the rewards
```C++
void claim( const account_name voter, const account_name bpname );
```
Parameters:
- voter : voter
- bpname : node name
  
logic:
1. compute voting account's recent vote age：last vote age+votes*(current block height-last settlement block height)
2. compute the node's total vote age：last vote age+votes*(current block height-last settlement block height)
3. compute rewards：node rewards pool amount*voter total vote age/node total vote age
4. add to voter account balance +=rewards
5. voting information vote age reset to zero，updat settlement block height
6. decrease node rewards pool -=rewards，total vote age decrease vote account vote age，update settlement block height


### block producton callback function    (system internal use)
```C++
void onblock(…const account_name bpname,const uint32_t schedule_version);
```
Parameters:
- bpname : node name
- schedule_version : node schedule version
  
logic:
1. bpname，producer.amount blocks produced increase 1
2. 9 EOS are rewarded every block produced，rewards will be added to BP balance according to reward ratio，then remaining rewards will be added to node's reward pool.
3. 1 EOS are rewarded to b1 (block.one) every block produced
4. refreshing elected 23 super nodes，select top 23 BP of all BPs according to votes，reset BPs by calling wasm interface set_proposed_producers,modify database.
 
### set emergency state
```C++
void setemergency( const account_name bpname, const bool emergency );
```

The blockchain will be set to emergency state aftedr more than 2/3 nodes agree，and all functions will be stopped immediately。
Parameters: 
- bpname : node 
- emergency : emergency state
 
## abi table
```json
"tables": [
        {
          "name":"accounts",
          "type":"account_info",
          "index_type": "i64",
          "key_names" : ["name"],
          "key_types" : ["account_name"]
       },{
          "name":"bps",
          "type":"bp_info",
          "index_type":"i64",
          "key_names": ["name"],
          "key_types": ["account_name"]
       },{
          "name":"votes",
          "type":"vote_info",
          "index_type":"i64",
          "key_names": ["bpname"],
          "key_types": ["account_name"]
       },{
          "name":"chainstatus",
          "type":"chain_status",
          "index_type":"i64",
          "key_names": ["name"],
          "key_types": ["account_name"]
       },{
          "name":"schedules",
          "type":"schedule_info",
          "index_type":"i64",
          "key_names": ["version"],
          "key_types": ["uint64"]
       }
  ]
```

## abi data structure
```json
"structs": [{
      "name": "transfer",
      "base": "",
      "fields": [
         {"name":"from",     "type":"account_name"},
         {"name":"to",       "type":"account_name"},
         {"name":"quantity", "type":"asset"},
         {"name":"memo",     "type":"string"}
      ]
    },{
      "name": "updatebp",
      "base": "",
      "fields": [
         {"name":"bpname",            "type":"account_name"},
         {"name":"block_signing_key", "type":"public_key"},
         {"name":"commission_rate",   "type":"uint32"},
         {"name":"url",               "type":"string"}
      ]
    },{
      "name": "unfreeze",
      "base": "",
      "fields": [
      {"name":"voter",      "type":"account_name"},
      {"name":"bpname",     "type":"account_name"}
      ]
    },{
      "name": "claim",
      "base": "",
      "fields": [
        {"name":"voter",      "type":"account_name"},
        {"name":"bpname",     "type":"account_name"}
      ]
    },{
      "name": "account_info",
      "base": "",
      "fields": [
        {"name":"name",         "type":"account_name"},
        {"name":"available",    "type":"asset"}
      ]
    },{
      "name": "vote_parameter",
      "base": "",
      "fields": [
        {"name":"voter",     "type":"account_name"},
        {"name":"bpname",    "type":"account_name"},
        {"name":"stake",     "type":"asset"}
      ]
    },{
      "name": "vote_info",
      "base": "",
      "fields": [
        {"name":"bpname",                "type":"account_name"},
        {"name":"staked",                "type":"asset"},
        {"name":"voteage",               "type":"int64"},
        {"name":"voteage_update_height", "type":"uint32"},
        {"name":"unstaking",             "type":"asset"},
        {"name":"unstake_height",        "type":"uint32"}
      ]
    },{
      "name": "bp_info",
      "base": "",
      "fields": [
        {"name":"name",                 "type":"account_name"},
        {"name":"block_signing_key",    "type":"public_key"},
        {"name":"commission_rate",      "type":"uint32"},
        {"name":"total_staked",         "type":"int64"},
        {"name":"rewards_pool",         "type":"asset"},
        {"name":"total_voteage",        "type":"int64"},
        {"name":"voteage_update_height","type":"uint32"},
        {"name":"url",                  "type":"string"},
        {"name":"emergency",            "type":"bool"}
      ]
    },{
      "name": "setemergency",
      "base": "",
      "fields": [
        {"name":"bpname",    "type":"account_name"},
        {"name":"emergency", "type":"bool"}
      ]
    },{
      "name": "chain_status",
      "base": "",
      "fields": [
        {"name":"name",       "type":"account_name"},
        {"name":"emergency",  "type":"bool"}
      ]
    },{
      "name": "producer",
      "base": "",
      "fields": [
        {"name":"bpname",  "type":"account_name"},
        {"name":"amount",  "type":"uint32"}
      ]
    },{
      "name": "schedule_info",
      "base": "",
      "fields": [
        {"name":"version",     "type":"uint64"},
        {"name":"block_height","type":"uint32"},
        {"name":"producers",   "type":"producer[]"}
      ]
    }
  ]
```