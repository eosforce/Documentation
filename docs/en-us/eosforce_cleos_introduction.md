# Cleos说明

cleos is a command line tool that interfaces with the REST API exposed by nodeos. In order to use cleos you will need to have the end point (IP address and port number) to a nodeos instance and also configure cleos to load the 'eosio::chain_api_plugin'. 


## test net

- IP: 47.98.249.86

- Port：8888

Eg: 

```bash
cleos -u http://47.98.249.86:8888 get info
```

## cleos command

```
 cleos [OPTIONS] SUBCOMMAND

Options:
  -h,--help                   Print this help message and exit
  -u,--url TEXT=http://localhost:8888/
                              the http/https URL where nodeos is running
  --wallet-url TEXT=http://localhost:8900/
                              the http/https URL where keosd is running
  -r,--header                 pass specific HTTP header; repeat this option to pass multiple headers
  -n,--no-verify              don't verify peer certificate when using HTTPS
  -v,--verbose                output verbose actions on error
  --print-request             print HTTP request to STDERR

Subcommands:
  version                     Retrieve version information
  create                      Create various items, on and off the blockchain
  get                         Retrieve various items and information from the blockchain
  set                         Set or update blockchain state
  transfer                    Transfer EOS from account to account
  net                         Interact with local p2p network connections
  wallet                      Interact with local wallet
  sign                        Sign a transaction
  push                        Push arbitrary transactions to the blockchain
  multisig                    Multisig contract commands
  system                      Send eosio.system contract action to the blockchain.
```
