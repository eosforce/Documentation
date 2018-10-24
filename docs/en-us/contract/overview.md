# Introduction

A real-world contract, simply stated, is an agreement governing outcomes for actions, given a set of inputs. A contract can range from formal legal contracts (e.g., a financial transaction) to something as simple as the "rules" of a game. Typical actions can be things such as fund transfers (in the case of a financial contract) or game moves (in the case of a game contract).

An EOSIO Smart Contract is software registered on the blockchain and executed on EOSIO nodes, that implements the semantics of a "contract" whose ledger of action requests are being stored on the blockchain. The Smart Contract defines the interface (actions, parameters, data structures) and the code that implements the interface. The code is compiled into a canonical bytecode format that nodes can retrieve and execute. The blockchain stores the transactions (e.g., legal transfers, game moves) of the contract. Each Smart Contract must be accompanied by a Ricardian Contract that defines the legally binding terms and conditions of the contract.

# Required Knowledge

C / C++ Experience

EOSIO based blockchains execute user-generated applications and code using WebAssembly (WASM). WASM is an emerging web standard with widespread support of Google, Microsoft, Apple, and others. At the moment the most mature toolchain for building applications that compile to WASM is clang/llvm with their C/C++ compiler. For best compatibility, it is recommended that you use the EOSIO toolchain.

Other toolchains in development by 3rd parties include: Rust, Python, and Solidity. While these other languages may appear simpler, their performance will likely impact the scale of application you can build. We expect that C++ will be the best language for developing high-performance and secure smart contracts and plan to use C++ for the foreseeable future.
Linux / Mac OS Experience

The EOSIO software supports the following environments:

    Amazon 2017.09 and higher
    Centos 7
    Fedora 25 and higher (Fedora 27 recommended)
    Mint 18
    Ubuntu 16.04 (Ubuntu 16.10 recommended)
    Ubuntu 18.04 LTS
    MacOS Darwin 10.12 and higher (MacOS 10.13.x recommended)

Command Line Knowledge

There are a variety of tools provided along with EOSIO which requires you to have basic command line knowledge in order to interact with.

# Communication Model

An EOSIO Smart Contract consists of a set of action and type definitions. Action definitions specify and implement the behaviors of the contract. The type definitions specify the required content and structures. EOSIO actions operate primarily in a message-based communication architecture. A client invokes actions by sending (pushing) messages to nodeos. This can be done using the cleos command. It can also be done using one of the EOSIO send methods (e.g., eosio::action::send). nodeos dispatches action requests to the WASM code that implements a contract. That code runs in its entirety, then processing continues to the next action.

EOSIO Smart Contracts can communicate with each other, e.g., to have another contract perform some operation pertinent to the completion of the current transaction, or to trigger a future transaction outside of the scope of the current transaction.

EOSIO supports two basic communication models, inline and deferred. An operation to perform within the current transaction is an example of an inline action, while a triggered future transaction is an example of a deferred action.

Communication among contracts should be considered as occurring asynchronously. The asynchronous communication model can result in spam, which the resource limiting algorithm will resolve.
Inline Communication

Inline communication takes the form of requesting other actions that need to be executed as part of the calling action. Inline actions operate with the same scopes and authorities of the original transaction, and are guaranteed to execute with the current transaction. These can effectively be thought of as nested transactions within the calling transaction. If any part of the transaction fails, the inline actions will unwind with the rest of the transaction. Calling the inline action generates no notification outside the scope of the transaction, regardless of success or failure.
Deferred Communication

Deferred communication conceptually takes the form of action notifications sent to a peer transaction. Deferred actions get scheduled to run, at best, at a later time, at the producer's discretion. There is no guarantee that a deferred action will be executed.

As already mentioned, deferred communication will get scheduled later at the producer's discretion. From the perspective of the originating transaction, i.e., the transaction that creates the deferred transaction, it can only determine whether the create request was submitted successfully or whether it failed (if it fails, it will fail immediately). Deferred transactions carry the authority of the contract that sends them. A transaction can cancel a deferred transaction.
Transactions VS. Actions

An action represents a single operation, whereas a transaction is a collection of one or more actions. A contract and an account communicate in the form of actions. Actions can be sent individually, or in combined form if they are intended to be executed as a whole.

Transaction with one action.

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
    ""
  ],
  "context_free_data": []
}

Transaction with multiple actions, these actions must all succeed or the transaction will fail.

{
  "expiration": "...",
  "region": 0,
  "ref_block_num": ...,
  "ref_block_prefix": ...,
  "net_usage_words": ..,
  "kcpu_usage": ..,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "...",
      "name": "...",
      "authorization": [{
          "actor": "...",
          "permission": "..."
        }
      ],
      "data": "..."
    }, {
      "account": "...",
      "name": "...",
      "authorization": [{
          "actor": "...",
          "permission": "..."
        }
      ],
      "data": "..."
    }
  ],
  "signatures": [
    ""
  ],
  "context_free_data": []
}

Context-Free Actions
Action Name Restrictions

Action types are actually base32 encoded 64-bit integers. This means they are limited to the characters a-z, 1-5, and '.' for the first 12 characters. If there is a 13th character then it is restricted to the first 16 characters ('.' and a-p).

For more detailed information, please see link here
Transaction Confirmations

On completion of the transaction, a transaction receipt is generated. This receipt takes the form a hash. Receiving a transaction hash does not mean that the transaction has been confirmed, it only means that the node accepted it without error, which also means that there is a high probability other producers will accept it.

By means of confirmation, you should see the transaction in the transaction history with the block number of which it is included.

Action Handlers and Action "Apply" Context

Smart contracts provide action handlers to do the work of requested actions. (More on this below) Each time an action runs, i.e., the action is "applied" by running the apply method in the contract implementation, EOSIO creates a new action "apply" context within which the action runs. The diagram below illustrates key elements of the action "apply" context.

![avatar](https://files.readme.io/6d71afc-action-apply-context-diagram.png)

From a global view of an EOSIO blockchain, every node in the EOSIO network gets a copy of and runs every action in every contract. Some of the nodes are doing the actual work of the contract, while others are processing in order to prove the validity of the transaction blocks. It is, therefore, important that contracts be able to determine "who they are", or basically, under which context are they running. Context identification information is provided in the action context, as illustrated in the above diagram by receiver, code, action. receiver is the account that is currently processing the action. code is the account that authorized the contract. action is the ID of the currently running action.

As discussed above, actions operate within transactions; if a transaction fails, the results of all actions in the transaction must be rolled back. A key part of the action context is the Current Transaction Data. This contains a transaction header, an ordered vector of all of the original actions in the transaction, a vector of the context free actions in the transaction, a prunable set of context free data (provided as a vector of blobs) defined by the code that implements the contract, and a full index to the vector of blobs.

Before processing an action, EOSIO sets up a clean working memory for the action. This is where the working variables for the action are held. An action's working memory is available only to that action, even for actions in the same transaction. Variables that might have been set when another action executed are not available within another action's context. The only way to pass state among actions is to persist it to and retrieve it from the EOSIO database. See Persistence API for details on how to use the EOSIO persistence services.

An action can have many side effects. Among these are:

    Change state persisted in the EOSIO persistent storage
    Notify the recipient of the current transaction
    Send inline action requests to a new receiver
    Generate new (deferred) transactions
    Cancel existing (in-flight) deferred transactions (i.e., cancel already-submitted deferred transaction requests)

Transaction Limitations

Every transaction must execute in 30ms or less. If a transaction contains several actions, and the sum of these actions is greater than 30ms, the entire transaction will fail. In situations without concurrency requirements for their actions this can be circumvented by including the CPU consuming actions in separate transactions.


# The Dispatcher Macro & Apply

Every smart contract must provide an apply action handler. The apply action handler is a function that listens to all incoming actions and performs the desired behavior. In order to respond to a particular action, code is required to identify and respond to specific actions requests. apply uses the receiver, code, and action input parameters as filters to map to the desired functions that implement particular actions. The apply function can filter on the code parameter using something like the following:

if (code == N(${contract_name}) {
   // your handler to respond to particular code
}

Within a given code, one can respond to a particular action by filtering on the action parameter. This is normally used in conjunction with the code filter.

if (action == N(${action_name}) {
    //your handler to respond to a particular action
}

The EOSIO_ABI macro

To simplify the work for contract developers, the EOSIO_ABI macro encapsulates the lower level action mapping details of the apply function, enabling developers to focus on their application implementation.

#define EOSIO_ABI( TYPE, MEMBERS ) \
extern "C" { \
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) { \
      auto self = receiver; \
      if( code == self ) { \
         TYPE thiscontract( self ); \
         switch( action ) { \
            EOSIO_API( TYPE, MEMBERS ) \
         } \
         /* does not allow destructor of thiscontract to run: eosio_exit(0); */ \
      } \
   } \
} \

A developer needs only to specify the code and action names from the contract in the macro, and all of the underlying C code mapping logic is generated by the macro. An example of use of the macro can be seen above, i.e., EOSIO_ABI( hello, (hi) ) where hello and hi are values from the contract.

In this example you can see there is one function, apply. All it does is log the actions delivered and makes no other checks. Anyone can deliver any action at any time provided the block producers allow it. Absent any required signatures, the contract will be billed for the bandwidth consumed.
apply

apply is the action handler, it listens to all incoming actions and reacts according to the specifications within the function. The apply function requires two input parameters, code and action.
code filter

In order to respond to a particular action, structure the apply function as follows. You may also construct a response to general actions by omitting the code filter.

if (code == N(${contract_name}) {
    // your handler to respond to particular code
}

You can also define responses to respective actions in the code block.
action filter

To respond to a particular action, structure your apply function as follows. This is normally used in conjuction with the code filter.

if (action == N(${action_name}) {
    //your handler to respond to a particular action
}

wast

Any program to be deployed to the EOSIO blockchain must be compiled into WASM format. This is the only format the blockchain accepts.

Once you have the CPP file ready, you can compile it into a text version of WASM (.wast) using the eosiocpp tool.

eosiocpp is deprecated from v1.2.0 and will be removed in v1.3.0 . It will be replaced into eosio-cpp of eosio.wasmsdk repository.
Parameters and arguments could be changed accordingly.

$ eosio-cpp -o ${contract}.wast ${contract}.cpp

abi

The Application Binary Interface (ABI) is a JSON-based description on how to convert user actions between their JSON and Binary representations. The ABI also describes how to convert the database state to/from JSON. Once you have described your contract via an ABI then developers and users will be able to interact with your contract seamlessly via JSON.

The ABI file can be generated from the .hpp files using the eosio-cpp tool by passing --abigen argument.

$ eosio-cpp -o ${contract}.wast ${contract}.cpp --abigen

The following is an example of what the skeleton contract ABI looks like:

{
  "types": [{
      "new_type_name": "account_name",
      "type": "name"
    }
  ],
  "structs": [{
      "name": "transfer",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "quantity": "uint64"
      }
    },{
      "name": "account",
      "base": "",
      "fields": {
        "account": "name",
        "balance": "uint64"
      }
    }
  ],
  "actions": [{
      "action": "transfer",
      "type": "transfer"
    }
  ],
  "tables": [{
      "table": "account",
      "type": "account",
      "index_type": "i64",
      "key_names" : ["account"],
      "key_types" : ["name"]
    }
  ]
}

You will notice that this ABI defines an action transfer of type transfer. This tells EOSIO that when ${account}->transfer action is seen that the payload is of type transfer. The type transfer is defined in the structs array in the object with name set to transfer.

  "structs": [{
      "name": "transfer",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "quantity": "uint64"
      }
    },{
...

The ABI has several fields, including from, to and quantity. These fields have the corresponding types account_name, and uint64. account_name is a built-in type used to represent base32 string as uint64. To see more about what built-in types are available, check here.

{
  "types": [{
      "new_type_name": "account_name",
      "type": "name"
    }
  ],
...

Inside the above types array we define a list of aliases for existing types. Here, we define name as an alias of account_name.



# Multi-Index DB API

Overview

EOSIO provides a set of services and interfaces that enable contract developers to
persist state across actions, and consequently transactions, boundaries. Without persistence, state that is generated during the processing of actions and transactions will be lost when processing goes out of scope. The persistence components include:

    Services to persist state in a database
    Enhanced query capabilities to find and retrieve database content
    C++ APIs to these services, intended for use by contract developers
    C APIs for access to core services, of interest to libraries and system developers

This document covers the first three topics.

The Need for Persistence Services

Actions perform the work of EOSIO contracts. Actions operate within an environment known as the action context. As illustrated in the Action "Apply" Context Diagram, an action context provides several things necessary for the execution of the action. One of those things is the action's working memory. This is where the action maintains its working state. Before processing an action, EOSIO sets up a clean working memory for the action. Variables that might have been set when another action executed are not available within the new action's context. The only way to pass state among actions is to persist it to and retrieve it from the EOSIO database.

The EOSIO Multi-Index API

The EOSIO Multi-Index API provides a C++ interface to the EOSIO database. The EOSIO Multi-Index API is patterned after Boost Multi-Index Containers. This API provides a model for object storage with rich retrieval capabilities, enabling the use of multiple indices with different sorting and access semantics. The Multi-Index API is provided by the eosio::multi_index C++ class found in the contracts/eosiolib folder of the EOSIO/eos GitHub repository. This class enables a contract written in C++ to read and modify persistent state in the EOSIO database.

The Multi-Index container interface eosio::multi_index provides a homogeneous container of an arbitrary C++ type (and it does not need to be a plain-old data type or be fixed-size) that is kept sorted in multiple indices by keys of various types that are derived from the objects. It can be compared to a traditional database table with rows, columns, and indices. It can also be easily compared to Boost Multi-index Containers. In fact many of the member function signatures of eosio::multi_index are modeled after boost::multi_index, although there are important differences.

eosio::multi_index can be conceptually viewed as tables in a conventional database in which the rows are the individual objects in the container, the columns are the member properties of the objects in the container, and the indices provide fast lookup of an object by a key compatible with an object member property.

Traditional database tables allow the index to be a user-defined function over some number of columns of the table. eosio::multi_index similarly allows the index to be any user-defined function (provided as a member function of the class/struct of the element type) but with its return value restricted to one of a limited set of supported key types.

Traditional database tables typically have a single unique primary key that allows
unambiguously identifying a particular row in the table and also provides the standard sort order for the rows in the table. eosio::multi_index supports a similar semantic, but the primary key of the object in the eosio::multi_index container must be a unique unsigned 64-bit integer. The objects in the eosio::multi_index container are sorted by the primary key index in ascending order of the unsigned 64-bit integer primary key.

EOSIO Multi-Index Iterators

A key differentiator of the EOSIO persistence services over other blockchain infrastructures is its Multi-Index iterators. Unlike some other blockchains that only provide a key-value store, EOSIO Multi-Index tables allow a contract developer to keep a collection of objects sorted by a variety of different key types, which could be derived from the data within the object. This enables rich retrieval capabilities. Up to 16 secondary indices can be defined, each having its own way of ordering and retrieving table contents.

The EOSIO Multi-Index iterators follow a pattern that is common to C++ iterators. All iterators are bi-directional const, either const_iterator or const_reverse_iterator. The iterators can be dereferenced to provide access to an object in the Multi-Index table.

Putting It All Together

How to Create Your EOSIO Multi-Index Table

Here is a summary of the steps to create your own persistent data using EOSIO Multi-Index tables.

    Define your object(s) using C++ class or struct. Each object will be in its own Multi-Index table.
    Define a const member function in the class/struct called primary_key that returns the uint64_t primary key value of your object.
    Determine the secondary indices. Up to 16 additional indices are supported. A secondary index
    supports several key types, listed below.
        idx64 - Primitive 64-bit unsigned integer key
        idx128 - Primitive 128-bit unsigned integer key, or a 128-bit fixed-size lexicographical key
        idx256 - 256-bit fixed-size lexicographical key
        idx_double - Double precision floating point key
        idx_long_double - Quadruple precision floating point key
    Define a key extractor for each secondary index. The key extractor is a function used to obtain the keys from the elements of the Multi-Index table. 

How to Use Your EOSIO Multi-Index Table

    Instantiate your Multi-Index table.
    Insert emplace into, and subsequently modify or erase objects in your table as required by your contract.
    Locate and traverse objects in your table using get, find and iterator operations.



# Naming Conventions

Standard Account Names

    Can only contain the characters .abcdefghijklmnopqrstuvwxyz12345. a-z (lowercase), 1-5 and . (period)
    Must start with a letter
    Must be 12 characters

Table Names, Structs, Functions, Classes

    Can only contain up to 12 alpha characters

Symbols

    Must be capitalized alpha characters between A and Z
    Must be 7 characters or less

Principle
Macro & functions related naming
N(base32 X)

Encode argument into EOSIO name. (base32)
Used for fixed name

If you want to convert into EOSIO name from variable, you should use eosio::string_to_name()
Tips

    To encode a script into EOSIO name, see eosio::string_to_name

    To decode from base32 and restore to string form, use eosio::to_string() (an alias of std::string)

    auto user_name_obj = eosio::name{user}; // account_name user
    std::string user_name = user_name_obj.to_string();




# [C++/C API](URLhttps://developers.eos.io/eosio-cpp/reference)

