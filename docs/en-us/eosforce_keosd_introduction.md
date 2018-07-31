# Keosd Overview

The program keosd, located in the eos/build/programs/keosd folder within the EOSIO/eos repository, can be used to store private keys that cleos will use to sign transactions sent to the block chain. keosd runs on your local machine and stores your private keys locally.

For most users the easiest way to use keosd is to have cleos launch it automatically. Wallet files (named foo.wallet for example) will also be created in this directory by default.

## Auto locking
By default keosd is set to auto lock your wallets after 15 minutes of inactivity. This is configurable in the config.ini. Be aware if you need to disable this feature you will have to set an enormous number -- setting it to 0 will cause keosd to always lock your wallet.

## Launching keosd manually
It is possible to launch keosd manually simply with

```
$ keosd
```
By default, keosd creates the folder ~/eosio-wallet and populates it with a basic config.ini file. The location of the config file can be specified on the command line using the --config-dir argument. The configuration file contains the http server endpoint for incoming http connections and other parameters for cross origin resource sharing. Be aware that if you allowed cleos to auto launch keosd, a config.ini will be generated that will be sightly different than if you had launched keosd manually.

The location of the wallet data folder can be specified on the command line using the --data-dir argument.

## Stopping keosd
The most effective way to stop keosd is to find the keosd process and send a SIGTERM signal to it. Note that because cleos auto-launches keosd, it is possible to end up with multiple instances of the keosd running. The following will find and terminate all instances.
```
$ pgrep keosd
3178
24991
$ pkill keosd
```
