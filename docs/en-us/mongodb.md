# eos mongoDB

## Overview
  eosforce v1.1.0 support mongoDB storage, replacing history api plugin，solving query ineffiency.

## Quick Configuration Start
1. start mongod, default port：27017。

2. config.ini
    - Add option:
    mongodb-uri = mongodb://127.0.0.1:27017/EOS

    - Add plugin：
    plugin = eosio::mongo_db_plugin

3. start nodeos. connect to mongo through mongodb-uri, create default database：EOS

## mongoDB collections：

- blocks    
- block_states  
- transactions  
- transaction_traces    
- action_traces 
- accounts  
- pub_keys  
- account_controls
