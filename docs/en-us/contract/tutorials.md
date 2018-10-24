# Getting Started with Smart Contracts

Step 1: Install Contract Development Toolkit (CDT)

EOSIO Contract Development Toolkit - Follow Installation instructions and proceed. The eosio-cpp utility that compiles contracts and generates ABI files is included in this toolkit.

First clone

	git clone --recursive https://github.com/eosio/eosio.cdt --branch v1.2.1 --single-branch
	cd eosio.cdt

Now run build.sh and provide the core symbol for the EOSIO blockchain that intend to deploy to. build.sh will install any dependencies that are needed.

	./build.sh <CORE_SYMBOL>

Finally, install the build

This install will install the core to /usr/local/eosio.cdt and symlinks to the top level tools (compiler, ld, etc.) to /usr/local/bin

	$ sudo ./install.sh

Step 2: Start Your Node

If you're using docker and you're container isn't running, run the following.

docker start eosio

You can start your own single-node blockchain with this single command if you're running nodeos locally.

	$ nodeos -e -p eosio --plugin eosio::chain_api_plugin \
        --plugin eosio::history_api_plugin

This command sets many flags and loads some optional plugins which we will need for the rest of this tutorial. Assuming everything worked properly, you should see a block generation message every 0.5 seconds.

On Docker,

	docker logs --tail 25 eosio

	...
	3165501ms thread-0   producer_plugin.cpp:944       produce_block        ] Produced block 00000a4c898956e0... #2636 @ 2018-05-25T16:52:45.500 signed by eosio [trxs: 0, lib: 2635, confirmed: 0]
	3166004ms thread-0   producer_plugin.cpp:944       produce_block        ] Produced block 00000a4d2d4a5893... #2637 @ 2018-05-25T16:52:46.000 signed by eosio [trxs: 0, lib: 2636, confirmed: 0]
	...

Step 3: Create a Wallet

A wallet is a repository of private keys necessary to authorize actions on the blockchain. These keys are stored on disk encrypted using a password generated for you. This password should be stored in a secure password manager or written down.

    Shell

	$ cleos wallet create --to-console
	Creating wallet: default
	Save password to use in the future to unlock this wallet.
	Without password imported keys will not be retrievable.
	"PW5JuBXoXJ8JHiCTXf...."
	
	$ cleos wallet create --to-console
	Creating wallet: default
	Save password to use in the future to unlock this wallet.
	Without password imported keys will not be retrievable.
	"PW5JuBXoXJ8JHiCTXf...."

Your wallet will automatically lock after a period of time, here's how you unlock it

    Text

	$ cleos wallet unlock
	password:
	
	$ cleos wallet unlock
	password:

For security purposes it is generally best to leave your wallet locked when you are not using it. To lock your wallet without shutting down nodeos you can do:

    Shell

	cleos wallet lock
	Locked: default
	
	cleos wallet lock
	Locked: default

You will need your wallet unlocked for the rest of this tutorial.
Load the Tutorial Key

The private blockchain launched in the steps above is created with a default initial key which must be loaded into the wallet (provided below)

    Shell

	$ cleos wallet import --private-key 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
	imported private key for: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
	
	$ cleos wallet import --private-key 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
	imported private key for: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV

Step 4: Load BIOS Contract

Now that we have a wallet with the key for the eosio account loaded, we can set a default system contract. For the purposes of development, the default eosio.bios contract can be used. This contract enables you to have direct control over the resource allocation of other accounts and to access other privileged API calls. In a public blockchain, this contract will manage the staking and unstaking of tokens to reserve bandwidth for CPU and network activity, and memory for contracts.

The eosio.bios contract can be found in the contracts/eosio.bios folder of your EOSIO source code. The command sequence below assumes it is being executed from the root of the EOSIO source, but you can execute it from anywhere by specifying the full path to ${EOSIO_SOURCE}/build/contracts/eosio.bios.

If you're using docker, the command is:

	$ cleos set contract eosio contracts/eosio.bios -p eosio@active
	Reading WAST...
	Assembling WASM...
	Publishing contract...
	executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
	#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
	#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...

If you built from source, the command is:

	$ cleos set contract eosio build/contracts/eosio.bios -p eosio@active
	Reading WAST...
	Assembling WASM...
	Publishing contract...
	executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
	#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
	#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...

The result of this command sequence is that cleos generated a transaction with two actions, eosio::setcode and eosio::setabi.

The code defines how the contract runs and the abi describes how to convert between binary and json representations of the arguments. While an abi is technically optional, all of the EOSIO tooling depends upon it for ease of use.

Any time you execute a transaction you will see output like:

	executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
	#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
	#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...

This can be read as: The action setcode as defined by eosio was executed by eosio contract with {args...}.

	#         ${executor} <= ${contract}:${action} ${args...}
	> console output from this execution, if any

As we will see in a bit, actions can be processed by more than one contract.

The last argument to this call was -p eosio@active. This tells cleos to sign this action with the active authority of the eosio account, i.e., to sign the action using the private key for the eosio account that we imported earlier.
Step 5: Create Accounts

Now that we have setup the basic system contract, we can start to create our own accounts. We will create two accounts, user and tester, and we will need to associate a key with each account. In this example, the same key will be used for both accounts.

To do this we first generate a key for the accounts.

	$ cleos create key --to-console
	Private key: 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
	Public key: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4

Then we import this key into our wallet:

	$ cleos wallet import --private-key 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
	imported private key for: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4

NOTE: Be sure to use the actual key value generated by the cleos command and not the one shown in the example above!

Keys are not automatically added to a wallet, so skipping this step could result in losing control of your account.
Create Two User Accounts

Next we will create two accounts, user and tester, using the key we created and imported above.

	$ cleos create account eosio user EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
	executed transaction: 8aedb926cc1ca31642ada8daf4350833c95cbe98b869230f44da76d70f6d6242  364 bytes  1000 cycles
	#         eosio <= eosio::newaccount            {"creator":"eosio","name":"user","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...
	
	$ cleos create account eosio tester EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
	executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083 366 bytes  1000 cycles
	#         eosio <= eosio::newaccount            {"creator":"eosio","name":"tester","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...

NOTE: The create account subcommand requires two keys, one for the OwnerKey (which in a production environment should be kept highly secure) and one for the ActiveKey. In this tutorial example, the same key is used for both.

Because we are using the eosio::history_api_plugin we can query all accounts that are controlled by our key:

$ cleos get accounts EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
{
  "account_names": [
    "tester",
    "user"
  ]
}



# Introduction to the EOSFORCEIO Token Contract

Eosio.token, Exchange, and Eosio.msig Contracts

This tutorial assumes that you have completed the tutorial Getting Started With Contracts.

At this stage the blockchain doesn't do much, so let's deploy the eosio.token contract. This contract enables the creation of many different tokens all running on the same contract but potentially managed by different users.

Before we can deploy the token contract we must create an account to deploy it to.

	$ cleos create account eosio eosio.token \
	        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 \
	...

Then we can deploy the contract which can be found in ${EOSIO_SOURCE}/build/contracts/eosio.token

	$ cleos set contract eosio.token build/contracts/eosio.token -p eosio.token@active
	Reading WAST...
	Assembling WASM...
	Publishing contract...
	executed transaction: 528bdbce1181dc5fd72a24e4181e6587dace8ab43b2d7ac9b22b2017992a07ad  8708 bytes  10000 cycles
	#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d0100000001ce011d60067f7e7f7f7f7f00...
	#         eosio <= eosio::setabi                {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...

Create the Currency Token

You can view the interface to eosio.token as defined by contracts/eosio.token/eosio.token.hpp:

   void create( account_name issuer,
                asset        maximum_supply );


   void issue( account_name to, asset quantity, string memo );

   void transfer( account_name from,
                  account_name to,
                  asset        quantity,
                  string       memo );

To create a new token we must call the create(...) action with the proper arguments. This command will use the symbol of the maximum supply to uniquely identify this token from other tokens. The issuer will be the one with authority to call issue and or perform other actions such as freezing, recalling, and whitelisting of owners.

The concise way to call this method, using positional arguments:

	$ cleos push action eosio.token create '[ "eosio", "1000000000.0000 SYS"]' \
	         -p eosio.token@active
	executed transaction: 0e49a421f6e75f4c5e09dd738a02d3f51bd18a0cf31894f68d335cd70d9c0e12  120 bytes  1000 cycles
	#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 SYS"}

Alternatively, a more verbose way to call this method, using named arguments:

	$ cleos push action eosio.token create \
	        '{"issuer":"eosio", "maximum_supply":"1000000000.0000 SYS"}' \
	        -p eosio.token@active
	executed transaction: 0e49a421f6e75f4c5e09dd738a02d3f51bd18a0cf31894f68d335cd70d9c0e12  120 bytes  1000 cycles
	#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 SYS"}

This command created a new token SYS with a precision of 4 decimals and a maximum supply of 1000000000.0000 SYS.

In order to create this token we required the permission of the eosio.token contract because it "owns" the symbol namespace (e.g. "SYS"). Future versions of this contract may allow other parties to buy symbol names automatically. For this reason we must pass -p eosio.token@active to authorize this call.
Issue Tokens to Account "User"

Now that we have created the token, the issuer can issue new tokens to the account user we created earlier. If you have not created an account named 'user', see the instructions here.

We will use the positional calling convention (vs named args).

	$ cleos push action eosio.token issue '[ "user", "100.0000 SYS", "memo" ]' \
	        -p eosio@active
	executed transaction: 822a607a9196112831ecc2dc14ffb1722634f1749f3ac18b73ffacd41160b019  268 bytes  1000 cycles
	#   eosio.token <= eosio.token::issue           {"to":"user","quantity":"100.0000 SYS","memo":"memo"}
	>> issue
	#   eosio.token <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}
	>> transfer
	#         eosio <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}
	#          user <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}

This time the output contains several different actions: one issue and three transfers. While the only action we signed was issue, the issue action performed an "inline transfer" and the "inline transfer" notified the sender and receiver accounts. The output indicates all of the action handlers that were called, the order they were called in, and whether or not any output was generated by the action.

Technically, the eosio.token contract could have skipped the inline transfer and opted to just modify the balances directly. However, in this case, the eosio.token contract is following our token convention that requires that all account balances be derivable by the sum of the transfer actions that reference them. It also requires that the sender and receiver of funds be notified so they can automate handling deposits and withdrawals.

If you want to see the actual transaction that was broadcast, you can use the -d -j options to indicate "don't broadcast" and "return transaction as json".

	$ cleos push action eosio.token issue '["user", "100.0000 SYS", "memo"]' -p eosio@active -d -j
	{
	  "expiration": "2018-05-25T19:02:58",
	  "ref_block_num": 18200,
	  "ref_block_prefix": 614206268,
	  "max_net_usage_words": 0,
	  "max_cpu_usage_ms": 0,
	  "delay_sec": 0,
	  "context_free_actions": [],
	  "actions": [{
	      "account": "eosio.token",
	      "name": "issue",
	      "authorization": [{
	          "actor": "eosio",
	          "permission": "active"
	        }
	      ],
	      "data": "00000000007015d640420f00000000000453595300000000046d656d6f"
	    }
	  ],
	  "transaction_extensions": [],
	  "signatures": [
	    "SIG_K1_Khyk1GsxWCx4axqYMF2AREDvaZtZdFaQifNPkR9DomR7toJ4sGua7pMBNq2osV5TY8rcGNcgNwn1eFe3noAXsoUA26HNDJ"
	  ],
	  "context_free_data": []
	}

Transfer Tokens to Account "Tester"

Now that account user has tokens, we will transfer some to account tester. We indicate that user authorized this action using the permission argument -p user@active.

	$ cleos push action eosio.token transfer \
	        '[ "user", "tester", "25.0000 SYS", "m" ]' -p user@active
	executed transaction: 06d0a99652c11637230d08a207520bf38066b8817ef7cafaab2f0344aafd7018  268 bytes  1000 cycles
	#   eosio.token <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}
	>> transfer
	#          user <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}
	#        tester <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}

Deploy Exchange Contract

Similar to the examples shown above, we can deploy the exchange contract. The exchange contract provides capabilities to create and trade currency. It is assumed this is being run from the root of the EOSIO source.

	$ cleos create account eosio exchange  \
	        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 \
	        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
	executed transaction: 4d38de16631a2dc698f1d433f7eb30982d855219e7c7314a888efbbba04e571c  364 bytes  1000 cycles
	#         eosio <= eosio::newaccount            {"creator":"eosio","name":"exchange","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxe...
	
	$ cleos set contract exchange build/contracts/exchange -p exchange@active
	Reading WAST/WASM from build/contracts/exchange/exchange.wasm...
	Using already assembled WASM...
	Publishing contract...
	executed transaction: 503dddec456ae301ef467c6a05bc6bf61e1ea21ab911ef6cc6e0750001b675c8  33888 bytes  5841 us
	#         eosio <= eosio::setcode               {"account":"exchange","vmtype":0,"vmversion":0,"code":"0061736d0100000001bb022f60067f7e7f7f7f7f00600...
	#         eosio <= eosio::setabi                {"account":"exchange","abi":"0e656f73696f3a3a6162692f312e30010c6163636f756e745f6e616d65046e616d650e0...

Deploy Eosio.msig Contract

The eosio.msig contract allows multiple parties to sign a single transaction asynchronously. EOSIO has multi-signature (multisig) support at a base level, but it requires a synchronous side channel where data is ferried around and signed. Eosio.msig is a more user friendly way of asynchronously proposing, approving and eventually publishing a transaction with multiple parties' consent.

The following steps can be used to deploy the eosio.msig contract.

	$ cleos create account eosio eosio.msig  \
	        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 \
	        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
	#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.msig","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJ...
	
	$ cleos set contract eosio.msig build/contracts/eosio.msig -p eosio.msig@active
	Reading WAST/WASM from build/contracts/eosio.msig/eosio.msig.wasm...
	Using already assembled WASM...
	Publishing contract...
	executed transaction: 3433c434bdef42ba2150d5df46c17e0258f20b7836a057911faa2daa66262338  8864 bytes  1319 us
	#         eosio <= eosio::setcode               {"account":"eosio.msig","vmtype":0,"vmversion":0,"code":"0061736d010000000198011760017f0060047f7e7e7...
	#         eosio <= eosio::setabi                {"account":"eosio.msig","abi":"0e656f73696f3a3a6162692f312e30030c6163636f756e745f6e616d65046e616d650...


# Multi Index Tables Example

Description

In this tutorial we will go through the steps to create and use Multi Index Tables in your smart contract.
Notes

Multi Index Tables are a way to cache state and/or data in RAM for fast access. Multi index tables support create, read, update and delete (CRUD) operations, something which the blockchain doesn't (it only supports create and read.)

Multi Index Tables provide a fast to access data store and are a practical way to store data for use in your smart contract. The blockchain records the transactions, but you should use Multi Index Tables to store application data.

They are multi index tables because they support using multiple indexes on the data, the primary index type must be uint64_t and must be unique, but the other, secondary, indexes can have duplicates. You can have up to 16 additional indexes and the field types can be uint64_t, uint128_t, uint256_t, double or long double

If you want to index on a string you will need to convert this to an integer type, and store the results in a field that you then index.
1. Create a struct

Create a struct which can be stored in the multi index table, and define getters on the fields you want to index.

Remember that one of these getters must be named "primary_key()", if you don't have this the compiler (eosio-cpp) will generate an error ... it can't find the field to use as the primary key.

If you want to have more than one index, (up to 16 are allowed) then define a getter for any field you want to index, at this point the name is less important as you will pass the getter name into the typedef.

    C++

      struct [[eosio::table]] mystruct 
      {
         uint64_t     key; 
         uint64_t     secondid;
         std::string  name; 
         std::string  account; 

         uint64_t primary_key() const { return key; } // getter for primary key
         uint64_t by_id() const {return secondid; } // getter for additional key
      };

      struct [[eosio::table]] mystruct 
      {
         uint64_t     key; 
         uint64_t     secondid;
         std::string  name; 
         std::string  account; 

         uint64_t primary_key() const { return key; } // getter for primary key
         uint64_t by_id() const {return secondid; } // getter for additional key
      };

Two additional things to note here:

    The attribute [[eosio::table]] is required for the ABI generator, eosio-cpp, to recognise that you want to expose this table via the ABI and make it visible outside the smart contract.

    The struct name is less than 12 characters and all in lower case.

2. typedef the multi index table and the define the indexes

Define the multi index table which will use mystruct, tell it what to index, and how to get the data which is being indexed. A primary key will automatically be created, so using the struct above if I want a multi index table with only a primary key I would define it as :

    C++

typedef eosio::multi_index<name(mystruct), mystruct> datastore;

typedef eosio::multi_index<name(mystruct), mystruct> datastore;

This defines the multi index passing in the tablename "name(mystruct)" and the struct name "mystruct". name(mystruct) performs a compile conversion of the struct name to a uint64_t and this uint64_t is used to identify data belonging to the multi index table.

To add additional or secondary indexes use the indexed_by template as a parameter, so the definition becomes

    C++

typedef eosio::multi_index<name(mystruct), mystruct, indexed_by<name(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>> datastore;

typedef eosio::multi_index<name(mystruct), mystruct, indexed_by<name(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>> datastore;

where

indexed_by<name(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>

the parameters

    the name of the field converted to an integer, name(secondid)
    a user defined key extractor, const_mem_fun<mystruct, uint64_t, &mystruct::by_id>

To have three indexes

    C++

      struct [[eosio::table]] mystruct 
      {
         uint64_t     key; 
         uint64_t     secondid;
         uint64_t			anotherid;
         std::string  name; 
         std::string  account; 

         uint64_t primary_key() const { return key; }
         uint64_t by_id() const {return secondid; }
         uint64_t by_anotherid() const {return anotherid; }
      };
      
typedef eosio::multi_index<name(mystruct), mystruct, indexed_by<name(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>, indexed_by<name(anotherid), const_mem_fun<mystruct, uint64_t, &mystruct::by_anotherid>>> datastore;
      
      
      

      struct [[eosio::table]] mystruct 
      {
         uint64_t     key; 
         uint64_t     secondid;
         uint64_t			anotherid;
         std::string  name; 
         std::string  account; 

         uint64_t primary_key() const { return key; }
         uint64_t by_id() const {return secondid; }
         uint64_t by_anotherid() const {return anotherid; }
      };
      
typedef eosio::multi_index<name(mystruct), mystruct, indexed_by<name(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>, indexed_by<name(anotherid), const_mem_fun<mystruct, uint64_t, &mystruct::by_anotherid>>> datastore;
      
      
      

and so on.

An important thing to note here is that struct name matches the table name, and that the the names that will appear in the abi file follow the rules (12 characters and all in lower case.) If they don't then the tables are not visible via the abi (you can get around this by editing the abi file.)
3. create local variables which are of the defined type

      // local instances of the multi indexes
      pollstable _polls;
      votes _votes;

Now I have defined a multi index table with two indexes and I can use this in my smart contract.

An example working smart contract using two multi index tables is shown below. Here you can see how to iterate over the tables and how to use two tables in the same contract.

    C++

	#include <eosiolib/eosio.hpp>
	
	using namespace eosio;
	
	class youvote : public contract {
	  public:
	      youvote(eosio::name s):contract(s), _polls(s, s), _votes(s, s)
	      {}
	
	      // public methods exposed via the ABI
	      // on pollsTable
	
	      [[eosio::action]]
	      void version()
	      {
	          print("YouVote version  0.01"); 
	
	      };
	      
	      [[eosio::action]]
	      void addpoll(eosio::name s, std::string pollName)
	      {
	          //require_auth(s);
	
	          print("Add poll ", pollName); 
	              
	          // update the table to include a new poll
	          _polls.emplace(get_self(), [&](auto& p)
	                                      {
	                                        p.key = _polls.available_primary_key();
	                                        p.pollId = _polls.available_primary_key();
	                                        p.pollName = pollName;
	                                        p.pollStatus = 0;
	                                        p.option = "";
	                                        p.count = 0;
	                                      });
	      };
	
	
	      [[eosio::action]]
	      void rmpoll(eosio::name s, std::string pollName)
	      {
	          //require_auth(s);
	
	          print("Remove poll ", pollName); 
	              
	          std::vector<uint64_t> keysForDeletion;
	          // find items which are for the named poll
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                  keysForDeletion.push_back(item.key);   
	              }
	          }
	          
	          // now delete each item for that poll
	          for (uint64_t key : keysForDeletion)
	          {
	              print("remove from _polls ", key);
	              auto itr = _polls.find(key);
	              if (itr != _polls.end())
	              {
	                _polls.erase(itr);
	              }
	          }
	
	
	          // add remove votes ... don't need it the actions are permanently stored on the block chain
	
	          std::vector<uint64_t> keysForDeletionFromVotes;
	          // find items which are for the named poll
	          for(auto& item : _votes)
	          {
	              if (item.pollName == pollName)
	              {
	                  keysForDeletionFromVotes.push_back(item.key);   
	              }
	          }
	          
	          // now delete each item for that poll
	          for (uint64_t key : keysForDeletionFromVotes)
	          {
	              print("remove from _votes ", key);
	              auto itr = _votes.find(key);
	              if (itr != _votes.end())
	              {
	                _votes.erase(itr);
	              }
	          }
	
	
	      };
	
	      [[eosio::action]]
	      void status(std::string pollName)
	      {
	          print("Change poll status ", pollName);
	
	          std::vector<uint64_t> keysForModify;
	          // find items which are for the named poll
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                  keysForModify.push_back(item.key);   
	              }
	          }
	          
	          // now get each item and modify the status
	          for (uint64_t key : keysForModify)
	          {
	
	            print("modify _polls status", key);
	            auto itr = _polls.find(key);
	            if (itr != _polls.end())
	            {
	              _polls.modify(itr, get_self(), [&](auto& p)
	                                              {
	                                                p.pollStatus = p.pollStatus + 1;
	                                              });
	            }
	          }
	      };
	
	      [[eosio::action]]
	      void statusreset(std::string pollName)
	      {
	          print("Reset poll status ", pollName); 
	              
	          std::vector<uint64_t> keysForModify;
	          // find all poll items
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                  keysForModify.push_back(item.key);   
	              }
	          }
	          
	          // update the status in each poll item
	          for (uint64_t key : keysForModify)
	          {
	              print("modify _polls status", key);
	              auto itr = _polls.find(key);
	              if (itr != _polls.end())
	              {
	                _polls.modify(itr, get_self(), [&](auto& p)
	                                                {
	                                                  p.pollStatus = 0;
	                                                });
	              }
	          }
	      };
	
	
	      [[eosio::action]]
	      void addpollopt(std::string pollName, std::string option)
	      {
	          print("Add poll option ", pollName, "option ", option); 
	
	          // find the pollId, from _polls, use this to update the _polls with a new option
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                    // can only add if the poll is not started or finished
	                    if(item.pollStatus == 0)
	                    {
	                        _polls.emplace(get_self(), [&](auto& p)
	                                          {
	                                            p.key = _polls.available_primary_key();
	                                            p.pollId = item.pollId;
	                                            p.pollName = item.pollName;
	                                            p.pollStatus = 0;
	                                            p.option = option;
	                                            p.count = 0;
	                                          });
	                    }
	                    else
	                    {
	                        print("Can not add poll option ", pollName, "option ", option, " Poll has started or is finished.");
	                    }
	
	                    break; // so you only add it once
	              }
	          }
	      };
	
	      [[eosio::action]]
	      void rmpollopt(std::string pollName, std::string option)
	      {
	          print("Remove poll option ", pollName, "option ", option); 
	              
	          std::vector<uint64_t> keysForDeletion;
	          // find and remove the named poll
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                  keysForDeletion.push_back(item.key);   
	              }
	          }
	          
	          
	          for (uint64_t key : keysForDeletion)
	          {
	              print("remove from _polls ", key);
	              auto itr = _polls.find(key);
	              if (itr != _polls.end())
	              {
	                  if (itr->option == option)
	                  {
	                      _polls.erase(itr);
	                  }
	              }
	          }
	      };
	
	
	      [[eosio::abi]]
	      void vote(std::string pollName, std::string option, std::string accountName)
	      {
	          print("vote for ", option, " in poll ", pollName, " by ", accountName); 
	
	          // is the poll open
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                  if (item.pollStatus != 1)
	                  {
	                      print("Poll ",pollName,  " is not open");
	                      return;
	                  }
	
	                  break; // only need to check status once
	              }
	          }
	
	          // has account name already voted?  
	          for(auto& vote : _votes)
	          {
	              if (vote.pollName == pollName && vote.account == accountName)
	              {
	                  print(accountName, " has already voted in poll ", pollName);
	                  //eosio_assert(true, "Already Voted");
	                  return;
	              }
	          }
	
	          uint64_t pollId =99999; // get the pollId for the _votes table
	
	          // find the poll and the option and increment the count
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName && item.option == option)
	              {
	                  pollId = item.pollId; // for recording vote in this poll
	
	                  _polls.modify(item, get_self(), [&](auto& p)
	                                                {
	                                                    p.count = p.count + 1;
	                                                });
	              }
	          }
	
	          // record that accountName has voted
	          _votes.emplace(get_self(), [&](auto& pv)
	                                      {
	                                        pv.key = _votes.available_primary_key();
	                                        pv.pollId = pollId;
	                                        pv.pollName = pollName;
	                                        pv.account = accountName;
	                                      });        
	      };
	
	  private:    
	
	    // create the multi index tables to store the data
	      struct [[eosio::table]] poll 
	      {
	        uint64_t      key; // primary key
	        uint64_t      pollId; // second key, non-unique, this table will have dup rows for each poll because of option
	        std::string   pollName; // name of poll
	        uint8_t      pollStatus =0; // staus where 0 = closed, 1 = open, 2 = finished
	        std::string  option; // the item you can vote for
	        uint32_t    count =0; // the number of votes for each itme -- this to be pulled out to separte table.
	
	        uint64_t primary_key() const { return key; }
	        uint64_t by_pollId() const {return pollId; }
	      };
	      typedef eosio::multi_index<N(poll), poll, indexed_by<N(pollId), const_mem_fun<poll, uint64_t, &poll::by_pollId>>> pollstable;
	
	      struct [[eosio::table]] pollvotes 
	      {
	         uint64_t     key; 
	         uint64_t     pollId;
	         std::string  pollName; // name of poll
	         std::string  account; //this account has voted, use this to make sure noone votes > 1
	
	         uint64_t primary_key() const { return key; }
	         uint64_t by_pollId() const {return pollId; }
	      };
	      typedef eosio::multi_index<name(pollvotes), pollvotes, indexed_by<name(pollId), const_mem_fun<pollvotes, uint64_t, &pollvotes::by_pollId>>> votes;
	
	      // local instances of the multi indexes
	      pollstable _polls;
	      votes _votes;
	};
	
	EOSIO_DISPATCH( youvote, (version)(addpoll)(rmpoll)(status)(statusreset)(addpollopt)(rmpollopt)(vote))
	
	#include <eosiolib/eosio.hpp>
	
	using namespace eosio;
	
	class youvote : public contract {
	  public:
	      youvote(eosio::name s):contract(s), _polls(s, s), _votes(s, s)
	      {}
	
	      // public methods exposed via the ABI
	      // on pollsTable
	
	      [[eosio::action]]
	      void version()
	      {
	          print("YouVote version  0.01"); 
	
	      };
	      
	      [[eosio::action]]
	      void addpoll(eosio::name s, std::string pollName)
	      {
	          //require_auth(s);
	
	          print("Add poll ", pollName); 
	              
	          // update the table to include a new poll
	          _polls.emplace(get_self(), [&](auto& p)
	                                      {
	                                        p.key = _polls.available_primary_key();
	                                        p.pollId = _polls.available_primary_key();
	                                        p.pollName = pollName;
	                                        p.pollStatus = 0;
	                                        p.option = "";
	                                        p.count = 0;
	                                      });
	      };
	
	
	      [[eosio::action]]
	      void rmpoll(eosio::name s, std::string pollName)
	      {
	          //require_auth(s);
	
	          print("Remove poll ", pollName); 
	              
	          std::vector<uint64_t> keysForDeletion;
	          // find items which are for the named poll
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                  keysForDeletion.push_back(item.key);   
	              }
	          }
	          
	          // now delete each item for that poll
	          for (uint64_t key : keysForDeletion)
	          {
	              print("remove from _polls ", key);
	              auto itr = _polls.find(key);
	              if (itr != _polls.end())
	              {
	                _polls.erase(itr);
	              }
	          }
	
	
	          // add remove votes ... don't need it the actions are permanently stored on the block chain
	
	          std::vector<uint64_t> keysForDeletionFromVotes;
	          // find items which are for the named poll
	          for(auto& item : _votes)
	          {
	              if (item.pollName == pollName)
	              {
	                  keysForDeletionFromVotes.push_back(item.key);   
	              }
	          }
	          
	          // now delete each item for that poll
	          for (uint64_t key : keysForDeletionFromVotes)
	          {
	              print("remove from _votes ", key);
	              auto itr = _votes.find(key);
	              if (itr != _votes.end())
	              {
	                _votes.erase(itr);
	              }
	          }
	
	
	      };
	
	      [[eosio::action]]
	      void status(std::string pollName)
	      {
	          print("Change poll status ", pollName);
	
	          std::vector<uint64_t> keysForModify;
	          // find items which are for the named poll
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                  keysForModify.push_back(item.key);   
	              }
	          }
	          
	          // now get each item and modify the status
	          for (uint64_t key : keysForModify)
	          {
	
	            print("modify _polls status", key);
	            auto itr = _polls.find(key);
	            if (itr != _polls.end())
	            {
	              _polls.modify(itr, get_self(), [&](auto& p)
	                                              {
	                                                p.pollStatus = p.pollStatus + 1;
	                                              });
	            }
	          }
	      };
	
	      [[eosio::action]]
	      void statusreset(std::string pollName)
	      {
	          print("Reset poll status ", pollName); 
	              
	          std::vector<uint64_t> keysForModify;
	          // find all poll items
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                  keysForModify.push_back(item.key);   
	              }
	          }
	          
	          // update the status in each poll item
	          for (uint64_t key : keysForModify)
	          {
	              print("modify _polls status", key);
	              auto itr = _polls.find(key);
	              if (itr != _polls.end())
	              {
	                _polls.modify(itr, get_self(), [&](auto& p)
	                                                {
	                                                  p.pollStatus = 0;
	                                                });
	              }
	          }
	      };
	
	
	      [[eosio::action]]
	      void addpollopt(std::string pollName, std::string option)
	      {
	          print("Add poll option ", pollName, "option ", option); 
	
	          // find the pollId, from _polls, use this to update the _polls with a new option
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                    // can only add if the poll is not started or finished
	                    if(item.pollStatus == 0)
	                    {
	                        _polls.emplace(get_self(), [&](auto& p)
	                                          {
	                                            p.key = _polls.available_primary_key();
	                                            p.pollId = item.pollId;
	                                            p.pollName = item.pollName;
	                                            p.pollStatus = 0;
	                                            p.option = option;
	                                            p.count = 0;
	                                          });
	                    }
	                    else
	                    {
	                        print("Can not add poll option ", pollName, "option ", option, " Poll has started or is finished.");
	                    }
	
	                    break; // so you only add it once
	              }
	          }
	      };
	
	      [[eosio::action]]
	      void rmpollopt(std::string pollName, std::string option)
	      {
	          print("Remove poll option ", pollName, "option ", option); 
	              
	          std::vector<uint64_t> keysForDeletion;
	          // find and remove the named poll
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                  keysForDeletion.push_back(item.key);   
	              }
	          }
	          
	          
	          for (uint64_t key : keysForDeletion)
	          {
	              print("remove from _polls ", key);
	              auto itr = _polls.find(key);
	              if (itr != _polls.end())
	              {
	                  if (itr->option == option)
	                  {
	                      _polls.erase(itr);
	                  }
	              }
	          }
	      };
	
	
	      [[eosio::abi]]
	      void vote(std::string pollName, std::string option, std::string accountName)
	      {
	          print("vote for ", option, " in poll ", pollName, " by ", accountName); 
	
	          // is the poll open
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName)
	              {
	                  if (item.pollStatus != 1)
	                  {
	                      print("Poll ",pollName,  " is not open");
	                      return;
	                  }
	
	                  break; // only need to check status once
	              }
	          }
	
	          // has account name already voted?  
	          for(auto& vote : _votes)
	          {
	              if (vote.pollName == pollName && vote.account == accountName)
	              {
	                  print(accountName, " has already voted in poll ", pollName);
	                  //eosio_assert(true, "Already Voted");
	                  return;
	              }
	          }
	
	          uint64_t pollId =99999; // get the pollId for the _votes table
	
	          // find the poll and the option and increment the count
	          for(auto& item : _polls)
	          {
	              if (item.pollName == pollName && item.option == option)
	              {
	                  pollId = item.pollId; // for recording vote in this poll
	
	                  _polls.modify(item, get_self(), [&](auto& p)
	                                                {
	                                                    p.count = p.count + 1;
	                                                });
	              }
	          }
	
	          // record that accountName has voted
	          _votes.emplace(get_self(), [&](auto& pv)
	                                      {
	                                        pv.key = _votes.available_primary_key();
	                                        pv.pollId = pollId;
	                                        pv.pollName = pollName;
	                                        pv.account = accountName;
	                                      });        
	      };
	
	  private:    
	
	    // create the multi index tables to store the data
	      struct [[eosio::table]] poll 
	      {
	        uint64_t      key; // primary key
	        uint64_t      pollId; // second key, non-unique, this table will have dup rows for each poll because of option
	        std::string   pollName; // name of poll
	        uint8_t      pollStatus =0; // staus where 0 = closed, 1 = open, 2 = finished
	        std::string  option; // the item you can vote for
	        uint32_t    count =0; // the number of votes for each itme -- this to be pulled out to separte table.
	
	        uint64_t primary_key() const { return key; }
	        uint64_t by_pollId() const {return pollId; }
	      };
	      typedef eosio::multi_index<N(poll), poll, indexed_by<N(pollId), const_mem_fun<poll, uint64_t, &poll::by_pollId>>> pollstable;
	
	      struct [[eosio::table]] pollvotes 
	      {
	         uint64_t     key; 
	         uint64_t     pollId;
	         std::string  pollName; // name of poll
	         std::string  account; //this account has voted, use this to make sure noone votes > 1
	
	         uint64_t primary_key() const { return key; }
	         uint64_t by_pollId() const {return pollId; }
	      };
	      typedef eosio::multi_index<name(pollvotes), pollvotes, indexed_by<name(pollId), const_mem_fun<pollvotes, uint64_t, &pollvotes::by_pollId>>> votes;
	
	      // local instances of the multi indexes
	      pollstable _polls;
	      votes _votes;
	};
	
	EOSIO_DISPATCH( youvote, (version)(addpoll)(rmpoll)(status)(statusreset)(addpollopt)(rmpollopt)(vote))

Note the EOSIO_ABI call, this exposes the functions via the abi, it's important the function names match the abi function name rules.


# Creating a Token on EOSFORCEIO

This tutorial assumes that you have completed the tutorial Getting Started With Contracts.

At this stage the blockchain doesn't do much, so let's deploy the eosio.token contract. This contract enables the creation of many different tokens all running on the same contract but potentially managed by different users.

Before we can deploy the token contract we must create an account to deploy it to.

	$ cleos create account eosio eosio.token EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 -p eosio@active
	...

Then we can deploy the contract which can be found in ${EOSIO_SOURCE}/build/contracts/eosio.token

	$ cleos set contract eosio.token build/contracts/eosio.token -p eosio.token
	Reading WAST...
	Assembling WASM...
	Publishing contract...
	executed transaction: 528bdbce1181dc5fd72a24e4181e6587dace8ab43b2d7ac9b22b2017992a07ad  8708 bytes  10000 cycles
	#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d0100000001ce011d60067f7e7f7f7f7f00...
	#         eosio <= eosio::setabi                {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...

Create the Currency Token

You can view the interface to eosio.token as defined by contracts/eosio.token/eosio.token.hpp:

   void create( account_name issuer,
                asset        maximum_supply );


   void issue( account_name to, asset quantity, string memo );

   void transfer( account_name from,
                  account_name to,
                  asset        quantity,
                  string       memo );

To create a new token we must call the create(...) action with the proper arguments. This command will use the symbol of the maximum supply to uniquely identify this token from other tokens. The issuer will be the one with authority to call issue and or perform other actions such as freezing, recalling, and whitelisting of owners.

The concise way to call this method, using positional arguments:

	$ cleos push action eosio.token create '[ "eosio", "1000000000.0000 SYS"]' -p eosio.token
	executed transaction: 0e49a421f6e75f4c5e09dd738a02d3f51bd18a0cf31894f68d335cd70d9c0e12  120 bytes  1000 cycles
	#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 SYS"}

Alternatively, a more verbose way to call this method, using named arguments:

	$ cleos push action eosio.token create '{"issuer":"eosio", "maximum_supply":"1000000000.0000 SYS"}' -p eosio.token@active
	executed transaction: 0e49a421f6e75f4c5e09dd738a02d3f51bd18a0cf31894f68d335cd70d9c0e12  120 bytes  1000 cycles
	#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 SYS"}

This command created a new token SYS with a pecision of 4 decimials and a maximum supply of 1000000000.0000 SYS.

In order to create this token we required the permission of the eosio.token contract because it "owns" the symbol namespace (e.g. "SYS"). Future versions of this contract may allow other parties to buy symbol names automatically. For this reason we must pass -p eosio.token to authorize this call.
Issue Tokens to Account "User"

Now that we have created the token, the issuer can issue new tokens to the account user we created earlier.

We will use the positional calling convention (vs named args).

	$ cleos push action eosio.token issue '[ "user", "100.0000 SYS", "memo" ]' -p eosio@active
	executed transaction: 822a607a9196112831ecc2dc14ffb1722634f1749f3ac18b73ffacd41160b019  268 bytes  1000 cycles
	#   eosio.token <= eosio.token::issue           {"to":"user","quantity":"100.0000 SYS","memo":"memo"}
	>> issue
	#   eosio.token <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}
	>> transfer
	#         eosio <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}
	#          user <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}

This time the output contains several different actions: one issue and three transfers. While the only action we signed was issue, the issue action performed an "inline transfer" and the "inline transfer" notified the sender and receiver accounts. The output indicates all of the action handlers that were called, the order they were called in, and whether or not any output was generated by the action.

Technically, the eosio.token contract could have skipped the inline transfer and opted to just modify the balances directly. However, in this case, the eosio.token contract is following our token convention that requires that all account balances be derivable by the sum of the transfer actions that reference them. It also requires that the sender and receiver of funds be notified so they can automate handling deposits and withdrawals.

If you want to see the actual transaction that was broadcast, you can use the -d -j options to indicate "don't broadcast" and "return transaction as json".

	$ cleos push action eosio.token issue '["user", "100.0000 SYS", "memo"]' -p eosio@active -d -j
	{
	  "expiration": "2018-04-01T15:20:44",
	  "region": 0,
	  "ref_block_num": 42580,
	  "ref_block_prefix": 3987474256,
	  "net_usage_words": 21,
	  "kcpu_usage": 1000,
	  "delay_sec": 0,
	  "context_free_actions": [],
	  "actions": [{
	      "account": "eosio.token",
	      "name": "issue",
	      "authorization": [{
	          "actor": "eosio",
	          "permission": "active"
	        }
	      ],
	      "data": "00000000007015d640420f000000000004454f5300000000046d656d6f"
	    }
	  ],
	  "signatures": [
	    "EOSJzPywCKsgBitRh9kxFNeMJc8BeD6QZLagtXzmdS2ib5gKTeELiVxXvcnrdRUiY3ExP9saVkdkzvUNyRZSXj2CLJnj7U42H"
	  ],
	  "context_free_data": []
	}

Transfer Tokens to Account "Tester"

Now that account user has tokens, we will transfer some to account tester. We indicate that user authorized this action using the permission argument -p user.

	$ cleos push action eosio.token transfer '[ "user", "tester", "25.0000 SYS", "m" ]' -p user@active
	executed transaction: 06d0a99652c11637230d08a207520bf38066b8817ef7cafaab2f0344aafd7018  268 bytes  1000 cycles
	#   eosio.token <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}
	>> transfer
	#          user <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}
	#        tester <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}




# Randomization in Contracts

By using openssl, sha256 and checksum256, a value can be produced that is deterministic and effectively random. This can be achieved by hashing (sha256) n number of hashes and their corresponding secrets and then selecting n number of values from the result. The determinism and randomness comes from having an outside source supply a hash and its corresponding secret.

In the example of two users wanting to play a game with 50/50 odds, each player must first submit a hash of their secret. By submitting their hash, they then become eligible to play the game. Once both players have submitted the hash of their secret, they are effectively engaged in the game. It's only when both players reveal their secrets that both of their submitted hashes and recently submitted secrets are hashed. From the result of this hash, two numbers are selected to determine a winner and a loser.
Step 1: Generate Secrets

    Shell

	openssl rand 32 -hex
	28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905
	
	openssl rand 32 -hex
	15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12
	
	openssl rand 32 -hex
	28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905
	
	openssl rand 32 -hex
	15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12

Step 2: Generate sha256 hashes

    Shell

	echo -n '28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905' | xxd -r -p | sha256sum -b | awk '{print $1}'
	d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883
	
	echo -n '15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12' | xxd -r -p | sha256sum -b | awk '{print $1}'
	50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129
	
	echo -n '28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905' | xxd -r -p | sha256sum -b | awk '{print $1}'
	d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883
	
	echo -n '15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12' | xxd -r -p | sha256sum -b | awk '{print $1}'
	50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129

Step 3: Submit Hash of Secret

Alice and Bob each submit their hash of their secret.

    Shell

	cleos push action chance submithash '[ "alice", "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883" ]' -p alice@active	
	
	cleos push action chance submithash '[ "bob", "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129" ]' -p bob@active
	
	cleos push action chance submithash '[ "alice", "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883" ]' -p alice@active	
	
	cleos push action chance submithash '[ "bob", "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129" ]' -p bob@active

Step 4: Submit Secret

Alice and Bob each submit the hash of their secret and the secret itself.

    Shell

	cleos push action chance submitboth '[ "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883", "28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905" ]' -p alice@active
	
	cleos push action chance submitboth '[ "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129", "15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12" ]' -p bob@active
	
	cleos push action chance submitboth '[ "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883", "28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905" ]' -p alice@active
	
	cleos push action chance submitboth '[ "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129", "15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12" ]' -p bob@active

Step 5: Conclusion

The simplest data type that can be used to store the secret and the hash of the secret is a struct.

    C++

	struct player {
	  checksum256 hash;
	  checksum256 secret;
	};
	
	struct player {
	  checksum256 hash;
	  checksum256 secret;
	};

When the players of the game are being instantiated, it is important to do so in succession. This is done so that the secrets and hashes for all the players are contiguously stored in memory.

    C++

	struct game {
		uint64_t id;
	  player player1;
	  player player2;
	}
	
	struct game {
		uint64_t id;
	  player player1;
	  player player2;
	}

Once the secret and the hash of the secret are received for all the players, the hashing of it all can be done.

    C++

	// variable to get result from hashing all players hashes and secrets
	checksum256 result;
	
	// hash the contents in memory, starting at game.player1 and spanning for 
	// sizeof(player)*2 bytes
	sha256( (char *)&game.player1, sizeof(player)*2, &result);
	
	// compares first and second 4 byte chunks in result to determine a winner
	int winner = result.hash[1] < result.hash[0] ? 0 : 1;
	
	// report appropriate winner
	if( winner ) {
	  report_winner(game.player1);
	} else {
	  report_winner(game.player2);
	}
	
	// variable to get result from hashing all players hashes and secrets
	checksum256 result;
	
	// hash the contents in memory, starting at game.player1 and spanning for 
	// sizeof(player)*2 bytes
	sha256( (char *)&game.player1, sizeof(player)*2, &result);
	
	// compares first and second 4 byte chunks in result to determine a winner
	int winner = result.hash[1] < result.hash[0] ? 0 : 1;
	
	// report appropriate winner
	if( winner ) {
	  report_winner(game.player1);
	} else {
	  report_winner(game.player2);
	}

It's important to note that sha256 is 32 bytes and that there are 8 chunks of uint32_t in it. In the above example, the first two are considered, but it could have been any of them.


# Upgrading the system contract

Indirect method using eosio.msig contract

Cleos currently provides tools to propose an action with the eosio.msig contract, but it does not provide an easy interface to propose a custom transaction.

So, at the moment it is difficult to propose an atomic transaction with multiple actions (for example eosio::setcode followed by eosio::setabi).

The advantage of the eosio.msig method is that it makes coordination much easier and does not place strict time limits (less than 9 hours) on signature collection.

The disadvantage of the eosio.msig method is that is requires the proposer to have sufficient RAM to propose the transaction and currently cleos does not provide convenient tools to use it with custom transactions like the one that would be necessary to atomically upgrade the system contract.

For now, it is recommended to use the direct method to upgrade the system contract.
Direct method (avoids using eosio.msig contract)

Each of the top 21 block producers should do the following:

    Get current system contract for later comparison (actual hash and ABI on the main-net blockchain will be different):

	$ programs/cleos/cleos get code -c original_system_contract.wast -a original_system_contract.abi eosio
	code hash: cc0ffc30150a07c487d8247a484ce1caf9c95779521d8c230040c2cb0e2a3a60
	saving wast to original_system_contract.wast
	saving abi to original_system_contract.abi

    Generate the unsigned transaction which upgrades the system contract:

	$ programs/cleos/cleos set contract -s -j -d eosio contracts/eosio.system | tail -n +4 > upgrade_system_contract_trx.json

The first few lines of the generated file should be something similar to (except with very different numbers for expiration, ref_block_num, and ref_block_prefix):

	{
	   "expiration": "2018-06-15T22:17:10",
	   "ref_block_num": 4552,
	   "ref_block_prefix": 511016679,
	   "max_net_usage_words": 0,
	   "max_cpu_usage_ms": 0,
	   "delay_sec": 0,
	   "context_free_actions": [],
	   "actions": [{
	      "account": "eosio",
	      "name": "setcode",
	      "authorization": [{
	          "actor": "eosio",
	          "permission": "active"
	        }
	      ],

and the last few lines should be:

	      }
	   ],
	   "transaction_extensions": [],
	   "signatures": [],
	   "context_free_data": []
	}

One of the top block producers should be chosen to lead the upgrade process. This lead producer should take their generated upgrade_system_contract_trx.json, rename it to upgrade_system_contract_official_trx.json, and do the following:

    Modify the expiration timestamp in upgrade_system_contract_official_trx.json to a time that is sufficiently far in the future to give enough time to collect all the necessary signatures, but not more than 9 hours from the time the transaction was generated. Also, keep in mind that the transaction will not be accepted into the blockchain if the expiration is more than 1 hour from the present time.

    Pass the upgrade_system_contract_official_trx.json file to all the other top 21 block producers.

Then each of the top 21 block producers should do the following:

    Compare their generated upgrade_system_contract_official_trx.json file with the upgrade_system_contract_official_trx.json provided by the lead producer. The only difference should be in expiration, ref_block_num, ref_block_prefix, for example:

	$ diff upgrade_system_contract_official_trx.json upgrade_system_contract_trx.json
	2,4c2,4
	<   "expiration": "2018-06-15T22:17:10",
	<   "ref_block_num": 4552,
	<   "ref_block_prefix": 511016679,
	---
	>   "expiration": "2018-06-15T21:20:39",
	>   "ref_block_num": 4972,
	>   "ref_block_prefix": 195390844,

    If the comparison is good, each block producer should proceed with signing the official upgrade transaction with the keys necessary to satisfy their active permission. If the block producer only has a single key (i.e the "active key") in the active permission of their block producing account, then they only need to generate one signature using that active key. This signing process can be done offline for extra security.

First, the block producer should collect all the necessary information. Let us assume that the block producers active key pair is (EOS5kBmh5kfo6c6pwB8j77vrznoAaygzoYvBsgLyMMmQ9B6j83i9c, 5JjpkhxAmEfynDgSn7gmEKEVcBqJTtu6HiQFf4AVgGv5A89LfG3). The block producer needs their active private key (5JjpkhxAmEfynDgSn7gmEKEVcBqJTtu6HiQFf4AVgGv5A89LfG3 in this example), the upgrade_system_contract_official_trx.json, and the chain_id (d0242fb30b71b82df9966d10ff6d09e4f5eb6be7ba85fd78f796937f1959315e in this example) which can be retrieved through cleos get info.

Then on a secure computer the producer can sign the transaction (the producer will need to paste in their private key when prompted):

	$ programs/cleos/cleos sign --chain-id d0242fb30b71b82df9966d10ff6d09e4f5eb6be7ba85fd78f796937f1959315e upgrade_system_contract_trx.json | tail -n 5
	private key:   "signatures": [
	    "SIG_K1_JzABB9gzDGwUHaRmox68UNcfxMVwMnEXqqS1MvtsyUX8KGTbsZ5aZQZjHD5vREQa5BkZ7ft8CceLBLAj8eZ5erZb9cHuy5"
	  ],
	  "context_free_data": []
	}

Make sure to use the chain_id of the actual main-net blockchain that the transaction will be submitted to and not the example chain_id provided above.

The output should include the signature (in this case "SIG_K1_JzABB9gzDGwUHaRmox68UNcfxMVwMnEXqqS1MvtsyUX8KGTbsZ5aZQZjHD5vREQa5BkZ7ft8CceLBLAj8eZ5erZb9cHuy5") which the producer should then send to the lead producer.

When the lead producer collects 15 producer signatures, the lead producer should do the following:

    Make a copy of the upgrade_system_contract_official_trx.json and call it upgrade_system_contract_official_trx_signed.json, and then modify the upgrade_system_contract_official_trx_signed.json so that the signatures field includes all 15 collected signatures. So the tail end of upgrade_system_contract_official_trx_signed.json could look something like:

	$ cat upgrade_system_contract_official_trx_signed.json | tail -n 20
	  "transaction_extensions": [],
	  "signatures": [
	   "SIG_K1_JzABB9gzDGwUHaRmox68UNcfxMVwMnEXqqS1MvtsyUX8KGTbsZ5aZQZjHD5vREQa5BkZ7ft8CceLBLAj8eZ5erZb9cHuy5",
	   "SIG_K1_Kj7XJxnPQSxEXZhMA8uK3Q1zAxp7AExzsRd7Xaa7ywcE4iUrhbVA3B6GWre5Ctgikb4q4CeU6Bvv5qmh9uJjqKEbbjd3sX",
	   "SIG_K1_KbE7qyz3A9LoQPYWzo4e6kg5ZVojQVAkDKuufUN2EwVUqtFhtjmGoC6QPQqLi8J7ftiysBp52wJBPjtNQUfZiGpGMsnZ1f",
	   "SIG_K1_KdQsE7ahHA9swE9SDGg4oF6XahpgHmZfEgQAy9KPBLd9HuwrF6c8m6jz43zizK2oo32Ejg1DYuMfoEvJgVfXo81jsqTHvA",
	   "SIG_K1_K6228Hi2z1WabgVdf5bk2UdKyyDSVFwkMaagTn9oLVDV8rCX7aQcjY94c39ah2CkLTsTEqzTPAYknJ8m2m9B7npPkHaFzc",
	   "SIG_K1_Jzdx75hBCA2WSaXgrupmrNbcQocUCsP8r1BKkPXMreiAKPZDwX9J3G8fS1HhyqWjc7FbukwZf8sVRdS3wKbJVpytqXe7Nn",
	   "SIG_K1_KW7Qu2SdPD3zuQKh2ziFLzn9QbKqeMpeiemULky5Bbg1Mst6ijbCX3k2AVFGNFLkNLA36PM1WAT5oipzu1B1K7ymRxTx1Z",
	   "SIG_K1_KXJf1KZNpz73YFKKE7u6jFgsQ8XcX3yA7rDX6ZmG1Qfnc9FLLmT1WViv4bwcPbxaEYfR6SNWfk5cCR9eao2si1soqkXq92",
	   "SIG_K1_JynjkHFT5UFGDpEcqdriXTzCGCwS36Xztq4UAWQHLQgRUZT2YFoLhUcc87kvUteqCUGVxsmSbfgWv1KLy24voKN4Qs5zTe",
	   "SIG_K1_JxhfCaGBhuNShpDHn7j1CryG3iSebvfi7FUnJsfkXNTiwLyq2NDBkeakwjCMWFbzr6qqWuMDLjfXbzdtU17f1wCXMjKSgk",
	   "SIG_K1_KcMSz89QG1ZRFNrXc69R63d5KXbJA8CBjNPYv1VEA3TRfjqVYuhyaHpGXQN4RSKDq4ygr3UTRYBQQVutkJnR6zZ4Ssgd7R",
	   "SIG_K1_JuxT6bhUAbDs6Q2ppuKyKauduvbaJLxvh4gBH4e4A9yRhvUBT7w3DcvMyhdaor27Kbu29jnqhTbvXcb57QqKWQDpboLv7e",
	   "SIG_K1_K8BuFYpCiC5FhpVK8ZAzc3VUg7vz6WwLoWBrGN6nnuqUjngGqvHp3UxDVzcwhqccHdv8kdPXvF6G1NszwF1dd3wjCrHBYw",
	   "SIG_K1_KfH5ZirPwDk1RQKvJv2AGPfsJyPXvXLegZ7LvcPmRtjtMiErs1STXLNT8kiBfhZr4xkWRA5NR1kMF3d49DFMJiB2iWMXJc",
	   "SIG_K1_KjJB8jtcqpVe3r5jouFiAa9wJeYqoLMh5xrUV6kBF6UWfbYjimMWBJWz2ZPomGDsk7JtdUESVrYj1AhYbdp3X48KLm5Cev"
	  ],
	  "context_free_data": []
	}

    Push the signed transaction to the blockchain:

	$ programs/cleos/cleos push transaction --skip-sign upgrade_system_contract_official_trx_signed.json
	{
	  "transaction_id": "202888b32e7a0f9de1b8483befac8118188c786380f6e62ced445f93fb2b1041",
	  "processed": {
	    "id": "202888b32e7a0f9de1b8483befac8118188c786380f6e62ced445f93fb2b1041",
	    "receipt": {
	      "status": "executed",
	      "cpu_usage_us": 4909,
	      "net_usage_words": 15124
	    },
	    "elapsed": 4909,
	    "net_usage": 120992,
	    "scheduled": false,
	    "action_traces": [{
	...

If you get an error message like the following:

Error 3090003: provided keys, permissions, and delays do not satisfy declared authorizations
Ensure that you have the related private keys inside your wallet and your wallet is unlocked.

That means that at least one of the signatures provided were bad. This may be because a producer signed the wrong transaction, used the wrong private key, or used the wrong chain ID.

If you get an error message like the following:

Error 3090002: irrelevant signature included
Please remove the unnecessary signature from your transaction!

That means unnecessary signatures were included. If there are 21 active producers, only signatures from exactly 15 of those 21 active producers are needed.

If you get an error message like the following:

Error 3040006: Transaction Expiration Too Far
Please decrease the expiration time of your transaction!

That means that the expiration time is more than 1 hour in the future and you need to wait some time before being allowed to push the transaction.

If you get an error message like the following:

Error 3040005: Expired Transaction
Please increase the expiration time of your transaction!

That means the expiration time of the signed transaction has passed and this entire process has to restart from step 1.

    Assuming the transaction successfully executes, everyone can then verify that the new contract is in place:

	$ programs/cleos/cleos get code -c new_system_contract.wast -a new_system_contract.abi eosio
	code hash: 9fd195bc5a26d3cd82ae76b70bb71d8ce83dcfeb0e5e27e4e740998fdb7b98f8
	saving wast to new_system_contract.wast
	saving abi to new_system_contract.abi
	$ diff original_system_contract.abi new_system_contract.abi
	584,592d583
	<         },{
	<           "name": "deferred_trx_id",
	<           "type": "uint32"
	<         },{
	<           "name": "last_unstake_time",
	<           "type": "time_point_sec"
	<         },{
	<           "name": "unstaking",
	<           "type": "asset"



# How to Write an ABI File

Introduction

Previously you deployed the eosio.token contract using the provided ABI file. This tutorial will overview how the ABI file correlates to the eosio.token contract.

ABI files can be generated using the eosio-cpp utility provided by eosio.cdt. However, there are several situations that may cause ABI's generation to malfunction or fail altogether. Advanced C++ patterns can trip it up and custom types can sometimes cause issues for ABI generation. For this reason, it's imperative you understand how ABI files work, so you can debug and fix if and when necessary.
What is an ABI?

The Application Binary Interface (ABI) is a JSON-based description on how to convert user actions between their JSON and Binary representations. The ABI also describes how to convert the database state to/from JSON. Once you have described your contract via an ABI then developers and users will be able to interact with your contract seamlessly via JSON.
Security Note

ABI can be bypassed when executing transactions. Messages and actions passed to a contract do not have to conform to the ABI. The ABI is a guide, not a gatekeeper.
Create an ABI File

Start with an empty ABI, name it eosio.token.abi

    Text

	{
	   "version": "eosio::abi/1.0",
	   "types": [],
	   "structs": [],
	   "actions": [],
	   "tables": [],
	   "ricardian_clauses": [],
	   "abi_extensions": [],
	   "___comment" : ""
	}
	
	
	{
	   "version": "eosio::abi/1.0",
	   "types": [],
	   "structs": [],
	   "actions": [],
	   "tables": [],
	   "ricardian_clauses": [],
	   "abi_extensions": [],
	   "___comment" : ""
	}

Types

An ABI enables any client or interface to interpret and even generate an GUI for you contract. For this to work in a consistent manner, describe the custom types that are used as a parameter in any public action or struct that needs to be described in the ABI.
Built-in Types

EOSIO implements a number of custom built-ins. Built-in types don't need to be described in an ABI file. If you would like to familiarize yourself with EOSIO's built-ins, they are defined here

Using eosio.token as an example, the only type that requires a description in the ABI file is account_name. The ABI uses "new_type_name" to describe explicit types, in this case account_name, and account_name is an alias of name type.

Below is the JSON to describe this type:

    JSON

	{
	   "new_type_name": "account_name",
	   "type": "name"
	}
	
	{
	   "new_type_name": "account_name",
	   "type": "name"
	}
	
	The ABI now looks like this:
	
	    JSON
	
	{
	   "version": "eosio::abi/1.0",
	   "types": [{
	     "new_type_name": "account_name",
	     "type": "name"
		 }],
	   "structs": [],
	   "actions": [],
	   "tables": [],
	   "ricardian_clauses": [],
	   "abi_extensions": []
	}
	
	{
	   "version": "eosio::abi/1.0",
	   "types": [{
	     "new_type_name": "account_name",
	     "type": "name"
		 }],
	   "structs": [],
	   "actions": [],
	   "tables": [],
	   "ricardian_clauses": [],
	   "abi_extensions": []
	}

Structs

Structs that are exposed to the ABI also need to be described. By looking at eosio.token.hpp, it can be quickly determined which structs are utilized by public actions. This is particularly important for the next step.

A struct's object definition in JSON looks like the following:

    JSON

	{
	   "name": "issue", //The name 
	   "base": "", 			//Inheritance, parent struct
	   "fields": []			//Array of field objects describing the struct's fields. 
	}
	
	{
	   "name": "issue", //The name 
	   "base": "", 			//Inheritance, parent struct
	   "fields": []			//Array of field objects describing the struct's fields. 
	}

Fields

    JSON

	{
	   "name":"", // The field's name
	   "type":""   // The field's type
	}    
	
	{
	   "name":"", // The field's name
	   "type":""   // The field's type
	}    

In the eosio.token contract, there's a number of structs that require definition. Please note, not all of the structs are explicitly defined, some correspond to an actions' parameters. Here's a list of structs that require an ABI description for the eosio.token contract:
Implicit Structs

The following structs are implicit in that a struct was never explicitly defined in the contract. Looking at the create action, you'll find two parameters, issuer of type account_name and maximum_supply of type asset. For brevity this tutorial won't break down every struct, but applying the same logic, you will end up with the following:
create

    JSON

	{
	  "name": "create",
	  "base": "",
	  "fields": [
	    {
	      "name":"issuer", 
	      "type":"account_name"
	    },
	    {
	      "name":"maximum_supply", 
	      "type":"asset"
	    }
	  ]
	}
	
	{
	  "name": "create",
	  "base": "",
	  "fields": [
	    {
	      "name":"issuer", 
	      "type":"account_name"
	    },
	    {
	      "name":"maximum_supply", 
	      "type":"asset"
	    }
	  ]
	}

issue

    JSON

	{
	  "name": "issue",
	  "base": "",
	  "fields": [
	    {
	      "name":"to", 
	      "type":"account_name"
	    },
	    {
	      "name":"quantity", 
	      "type":"asset"
	    },
	    {
	      "name":"memo", 
	      "type":"string"
	    }
	  ]
	}
	
	{
	  "name": "issue",
	  "base": "",
	  "fields": [
	    {
	      "name":"to", 
	      "type":"account_name"
	    },
	    {
	      "name":"quantity", 
	      "type":"asset"
	    },
	    {
	      "name":"memo", 
	      "type":"string"
	    }
	  ]
	}

retire

    JSON

	{
	  "name": "retire",
	  "base": "",
	  "fields": [
	    {
	      "name":"quantity", 
	      "type":"asset"
	    },
	    {
	      "name":"memo", 
	      "type":"string"
	    }
	  ]
	}
	
	{
	  "name": "retire",
	  "base": "",
	  "fields": [
	    {
	      "name":"quantity", 
	      "type":"asset"
	    },
	    {
	      "name":"memo", 
	      "type":"string"
	    }
	  ]
	}

transfer

    JSON

	{
	  "name": "transfer",
	  "base": "",
	  "fields": [
	    {
	      "name":"from", 
	      "type":"account_name"
	    },
	    {
	      "name":"to", 
	      "type":"account_name"
	    },
	    {
	      "name":"quantity", 
	      "type":"asset"
	    },
	    {
	      "name":"memo", 
	      "type":"string"
	    }
	  ]
	}
	
	{
	  "name": "transfer",
	  "base": "",
	  "fields": [
	    {
	      "name":"from", 
	      "type":"account_name"
	    },
	    {
	      "name":"to", 
	      "type":"account_name"
	    },
	    {
	      "name":"quantity", 
	      "type":"asset"
	    },
	    {
	      "name":"memo", 
	      "type":"string"
	    }
	  ]
	}

close

    JSON

	{
	  "name": "close",
	  "base": "",
	  "fields": [
	    {
	      "name":"owner", 
	      "type":"account_name"
	    },
	    {
	      "name":"symbol", 
	      "type":"symbol"
	    }
	  ]
	 }
	
	{
	  "name": "close",
	  "base": "",
	  "fields": [
	    {
	      "name":"owner", 
	      "type":"account_name"
	    },
	    {
	      "name":"symbol", 
	      "type":"symbol"
	    }
	  ]
	}

Explicit Structs

These structs are explicitly defined, as they are a requirement to instantiate a multi-index table. Describing them is no different than defining the implicit structs as demonstrated above.
account

    JSON

	{
	  "name": "account",
	  "base": "",
	  "fields": [
	    {
	      "name":"balance", 
	      "type":"asset"
	    }
	  ]
	}
	
	{
	  "name": "account",
	  "base": "",
	  "fields": [
	    {
	      "name":"balance", 
	      "type":"asset"
	    }
	  ]
	}

currency_stats

    JSON

	{
	  "name": "currency_stats",
	  "base": "",
	  "fields": [
	    {
	      "name":"supply", 
	      "type":"asset"
	    },
	    {
	      "name":"max_supply", 
	      "type":"asset"
	    },
	    {
	      "name":"issuer", 
	      "type":"account_name"
	    }
	  ]
	}
	
	{
	  "name": "currency_stats",
	  "base": "",
	  "fields": [
	    {
	      "name":"supply", 
	      "type":"asset"
	    },
	    {
	      "name":"max_supply", 
	      "type":"asset"
	    },
	    {
	      "name":"issuer", 
	      "type":"account_name"
	    }
	  ]
	}

Actions

An action's JSON object definition looks like the following:

    JSON

	{
	  "name": "transfer", 			//The name of the action as defined in the contract
	  "type": "transfer", 			//The name of the implicit struct as described in the ABI
	  "ricardian_contract": "" 	//An optional ricardian clause to associate to this action describing its intended functionality.
	}
	
	{
	  "name": "transfer", 			//The name of the action as defined in the contract
	  "type": "transfer", 			//The name of the implicit struct as described in the ABI
	  "ricardian_contract": "" 	//An optional ricardian clause to associate to this action describing its intended functionality.
	}

Describe the actions of the eosio.token contract by aggregating all the public functions described in the eosio.token contract's header file.

Then describe each action's type according to its previously described struct. In most situations, the function name and the struct name will be equal, but are not required to be equal.

Below is a list of actions that link to their source code with example JSON provided for how each action would be described.
create

    JSON

	{
	  "name": "create",
	  "type": "create",
	  "ricardian_contract": ""
	}
	
	{
	  "name": "create",
	  "type": "create",
	  "ricardian_contract": ""
	}

issue

    JSON

	{
	  "name": "issue",
	  "type": "issue",
	  "ricardian_contract": ""
	}
	
	{
	  "name": "issue",
	  "type": "issue",
	  "ricardian_contract": ""
	}

retire

    JSON

	{
	  "name": "retire",
	  "type": "retire",
	  "ricardian_contract": ""
	}
	
	{
	  "name": "retire",
	  "type": "retire",
	  "ricardian_contract": ""
	}

transfer

    JSON

	{
	  "name": "transfer",
	  "type": "transfer",
	  "ricardian_contract": ""
	}
	
	{
	  "name": "transfer",
	  "type": "transfer",
	  "ricardian_contract": ""
	}

close

    JSON

	{
	  "name": "close",
	  "type": "close",
	  "ricardian_contract": ""
	}
	
	{
	  "name": "close",
	  "type": "close",
	  "ricardian_contract": ""
	}

Tables

Describe the tables. Here's a table's JSON object definition:

    JSON

	{
	  "name": "",       //The name of the table, determined during instantiation. 
	  "type": "", 			//The table's corresponding struct
	  "index_type": "", //The type of primary index of this table
	  "key_names" : [], //An array of key names, length must equal length of key_types member
	  "key_types" : []  //An array of key types that correspond to key names array member, length of array must equal length of key names array.
	}
	
	{
	  "name": "",       //The name of the table, determined during instantiation. 
	  "type": "", 			//The table's corresponding struct
	  "index_type": "", //The type of primary index of this table
	  "key_names" : [], //An array of key names, length must equal length of key_types member
	  "key_types" : []  //An array of key types that correspond to key names array member, length of array must equal length of key names array.
	}

The eosio.token contract instantiates two tables, accounts and stat.

The accounts table is an i64 index, based on the account struct, has a uint64 as it's primary key and it's key been arbitrarily named "currency".

Here's how the accounts table would be described in the ABI

    JSON

	{
	  "name": "accounts",
	  "type": "account", // Corresponds to previously defined struct
	  "index_type": "i64",
	  "key_names" : ["currency"],
	  "key_types" : ["uint64"]
	}
	
	{
	  "name": "accounts",
	  "type": "account", // Corresponds to previously defined struct
	  "index_type": "i64",
	  "key_names" : ["currency"],
	  "key_types" : ["uint64"]
	}

The stat table is an i64 index, based on the currenct_stats struct, has a uint64 as it's primary key and it's key been arbitrarily named "currency"

Here's how the stat table would be described in the ABI

    JSON

	{
	  "name": "stat",
	  "type": "currency_stats",
	  "index_type": "i64",
	  "key_names" : ["currency"],
	  "key_types" : ["uint64"]
	}
	
	{
	  "name": "stat",
	  "type": "currency_stats",
	  "index_type": "i64",
	  "key_names" : ["currency"],
	  "key_types" : ["uint64"]
	}

You'll notice the above tables have the same "key name." Naming your keys similar names is symbolic in that it can potentially suggest a subjective relationship. As with this implementation, implying that any given value can be used to query different tables.
Putting it all Together

Finally, an ABI file that accurately describes the eosio.token contract.

    JSON

	{
	  "version": "eosio::abi/1.0",
	  "types": [
	    {
	      "new_type_name": "account_name",
	      "type": "name"
	    }
	  ],
	  "structs": [
	    {
	      "name": "create",
	      "base": "",
	      "fields": [
	        {
	          "name":"issuer", 
	          "type":"account_name"
	        },
	        {
	          "name":"maximum_supply", 
	          "type":"asset"
	        }
	      ]
	    },
	    {
	       "name": "issue",
	       "base": "",
	       "fields": [
	          {
	            "name":"to", 
	            "type":"account_name"
	          },
	          {
	            "name":"quantity", 
	            "type":"asset"
	          },
	          {
	            "name":"memo", 
	            "type":"string"
	          }
	       ]
	    },
	    {
	       "name": "retire",
	       "base": "",
	       "fields": [
	          {
	            "name":"quantity", 
	            "type":"asset"
	          },
	          {
	            "name":"memo", 
	            "type":"string"
	          }
	       ]
	    },
	    {
	       "name": "close",
	       "base": "",
	       "fields": [
	          {
	            "name":"owner", 
	            "type":"account_name"
	          },
	          {
	            "name":"symbol", 
	            "type":"symbol"
	          }
	       ]
	    },
	    {
	      "name": "transfer",
	      "base": "",
	      "fields": [
	        {
	          "name":"from", 
	          "type":"account_name"
	        },
	        {
	          "name":"to", 
	          "type":"account_name"
	        },
	        {
	          "name":"quantity", 
	          "type":"asset"
	        },
	        {
	          "name":"memo", 
	          "type":"string"
	        }
	      ]
	    },
	    {
	      "name": "account",
	      "base": "",
	      "fields": [
	        {
	          "name":"balance", 
	          "type":"asset"
	        }
	      ]
	    },
	    {
	      "name": "currency_stats",
	      "base": "",
	      "fields": [
	        {
	          "name":"supply", 
	          "type":"asset"
	        },
	        {
	          "name":"max_supply", 
	          "type":"asset"
	        },
	        {
	          "name":"issuer", 
	          "type":"account_name"
	        }
	      ]
	    }
	  ],
	  "actions": [
	    {
	      "name": "transfer",
	      "type": "transfer",
	      "ricardian_contract": ""
	    },
	    {
	      "name": "issue",
	      "type": "issue",
	      "ricardian_contract": ""
	    },
	    {
	      "name": "retire",
	      "type": "retire",
	      "ricardian_contract": ""
	    },
	    {
	      "name": "create",
	      "type": "create",
	      "ricardian_contract": ""
	    },
	    {
	      "name": "close",
	      "type": "close",
	      "ricardian_contract": ""
	    }
	  ],
	  "tables": [
	    {
	      "name": "accounts",
	      "type": "account",
	      "index_type": "i64",
	      "key_names" : ["currency"],
	      "key_types" : ["uint64"]
	    },
	    {
	      "name": "stat",
	      "type": "currency_stats",
	      "index_type": "i64",
	      "key_names" : ["currency"],
	      "key_types" : ["uint64"]
	    }
	  ],
	  "ricardian_clauses": [],
	  "abi_extensions": []
	}
	
	{
	  "version": "eosio::abi/1.0",
	  "types": [
	    {
	      "new_type_name": "account_name",
	      "type": "name"
	    }
	  ],
	  "structs": [
	    {
	      "name": "create",
	      "base": "",
	      "fields": [
	        {
	          "name":"issuer", 
	          "type":"account_name"
	        },
	        {
	          "name":"maximum_supply", 
	          "type":"asset"
	        }
	      ]
	    },
	    {
	       "name": "issue",
	       "base": "",
	       "fields": [
	          {
	            "name":"to", 
	            "type":"account_name"
	          },
	          {
	            "name":"quantity", 
	            "type":"asset"
	          },
	          {
	            "name":"memo", 
	            "type":"string"
	          }
	       ]
	    },
	    {
	       "name": "retire",
	       "base": "",
	       "fields": [
	          {
	            "name":"quantity", 
	            "type":"asset"
	          },
	          {
	            "name":"memo", 
	            "type":"string"
	          }
	       ]
	    },
	    {
	       "name": "close",
	       "base": "",
	       "fields": [
	          {
	            "name":"owner", 
	            "type":"account_name"
	          },
	          {
	            "name":"symbol", 
	            "type":"symbol"
	          }
	       ]
	    },
	    {
	      "name": "transfer",
	      "base": "",
	      "fields": [
	        {
	          "name":"from", 
	          "type":"account_name"
	        },
	        {
	          "name":"to", 
	          "type":"account_name"
	        },
	        {
	          "name":"quantity", 
	          "type":"asset"
	        },
	        {
	          "name":"memo", 
	          "type":"string"
	        }
	      ]
	    },
	    {
	      "name": "account",
	      "base": "",
	      "fields": [
	        {
	          "name":"balance", 
	          "type":"asset"
	        }
	      ]
	    },
	    {
	      "name": "currency_stats",
	      "base": "",
	      "fields": [
	        {
	          "name":"supply", 
	          "type":"asset"
	        },
	        {
	          "name":"max_supply", 
	          "type":"asset"
	        },
	        {
	          "name":"issuer", 
	          "type":"account_name"
	        }
	      ]
	    }
	  ],
	  "actions": [
	    {
	      "name": "transfer",
	      "type": "transfer",
	      "ricardian_contract": ""
	    },
	    {
	      "name": "issue",
	      "type": "issue",
	      "ricardian_contract": ""
	    },
	    {
	      "name": "retire",
	      "type": "retire",
	      "ricardian_contract": ""
	    },
	    {
	      "name": "create",
	      "type": "create",
	      "ricardian_contract": ""
	    },
	    {
	      "name": "close",
	      "type": "close",
	      "ricardian_contract": ""
	    }
	  ],
	  "tables": [
	    {
	      "name": "accounts",
	      "type": "account",
	      "index_type": "i64",
	      "key_names" : ["currency"],
	      "key_types" : ["uint64"]
	    },
	    {
	      "name": "stat",
	      "type": "currency_stats",
	      "index_type": "i64",
	      "key_names" : ["currency"],
	      "key_types" : ["uint64"]
	    }
	  ],
	  "ricardian_clauses": [],
	  "abi_extensions": []
	}

Cases not Covered by Token Contract
Vectors

When describing a vector in your ABI file, simply append the type with [], so if you need to describe a vector of permission levels, you would describe it like so: permission_level[]
Struct Base

It's a rarely used property worth mentioning. You can use base ABI struct property to reference another struct for inheritance, as long as that struct is also described in the same ABI file. Base will do nothing or potentially throw an error if your smart contract logic does not support inheritance.

You can see an example of base in use in the system contract source code and ABI
Extra ABI Properties Not Covered Here

A few properties of the ABI specification were skipped here for brevity, however, there is a pending ABI specification that will outline every property of the ABI in its entirety.
Ricardian Clauses

Ricardian clauses describe the intended outcome of a particular actions. It may also be utilized to establish terms between the sender and the contract.
ABI Extensions

A generic "future proofing" layer that allows old clients to skip the parsing of "chunks" of extension data. For now, this property is unused. In the future each extension would have its own "chunk" in that vector so that older clients skip it and newer clients that understand how to interpret it.
Maintenance

Every time you change a struct, add a table, add an action or add parameters to an action, use a new type, you will need to remember to update your ABI file. In many cases failure to update your ABI file will not produce any error.
Troubleshooting
Table returns no rows

Check that your table is accurately described in the GLOSSARY:ABI file. For example, If you use cleos to add a table on a contract with a malformed GLOSSARY:ABI definition and then get rows from that table, you will recieve an empty result. cleos will not produce an error when adding a row nor reading a row when a contract has failed to properly describe its tables in its GLOSSARY:ABI File.
