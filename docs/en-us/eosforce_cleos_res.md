# Cleos reference

## cleos create
```
cleos create SUBCOMMAND

Subcommands:
  key                         Create a new keypair and print the public and private keys
  account  Create an account, buy ram, stake for bandwidth for the account
```

## cleos get 
```
cleos get SUBCOMMAND

Subcommands:
  info                        Get current blockchain information
  block                       Retrieve a full block from the blockchain
  account                     Retrieve an account from the blockchain
  code                        Retrieve the code and ABI for an account
  abi                         Retrieve the ABI for an account
  table                       Retrieve the contents of a database table
  currency                    Retrieve information related to standard currencies
  accounts                    Retrieve accounts associated with a public key
  servants                    Retrieve accounts which are servants of a given account
  transaction                 Retrieve a transaction from the blockchain
  actions                     Retrieve all actions with specific account name referenced in authorization or receiver
```
## cleos set
```
cleos set SUBCOMMAND

Subcommands:
  code                        Create or update the code on an account
  abi                         Create or update the abi on an account
  contract                    Create or update the contract on an account
  account                     set or update blockchain account state
  action                      set or update blockchain action state
```
## cleos net
```
 cleos net SUBCOMMAND

Subcommands:
  connect                     start a new connection to a peer
  disconnect                  close an existing connection
  status                      status of existing connection
  peers                       status of all existing peers
  ```

## cleos wallet
```
cleos wallet SUBCOMMAND

Subcommands:
  create                      Create a new wallet locally
  open                        Open an existing wallet
  lock                        Lock wallet
  lock_all                    Lock all unlocked wallets
  unlock                      Unlock wallet
  import                      Import private key into wallet
  create_key                  Create private key within wallet
  list                        List opened wallets, * = unlocked
  keys                        List of public keys from all unlocked wallets.
  private_keys                List of private keys from an unlocked wallet in wif or PVT_R1 format.
  stop                        Stop keosd (doesn't work with nodeos).
```

## cleos sign
```
cleos sign [OPTIONS] transaction

Positionals:
  transaction TEXT            The JSON string or filename defining the transaction to sign (required)

Options:
  -k,--private-key TEXT       The private key that will be used to sign the transaction
  -c,--chain-id TEXT          The chain id that will be used to sign the transaction
  -p,--push-transaction       Push transaction after signing
  ```

  ## cleos push
  ```
  cleos push SUBCOMMAND

Subcommands:
  action                      Push a transaction with a single action
  transaction                 Push an arbitrary JSON transaction
  transactions                Push an array of arbitrary JSON transactions
  ```

  ## cleos msig
  ```
  cleos multisig SUBCOMMAND

Subcommands:
  propose                     Propose transaction
  review                      Review transaction
  approve                     Approve proposed transaction
  unapprove                   Unapprove proposed transaction
  cancel                      Cancel proposed transaction
  exec                        Execute proposed transaction
  ```

  > cleos transfer  Transfer EOS from account to account, invalid in eosforce;

  > cleos system    Send eosio.system contract action to the blockchain，invalid in eosforce;