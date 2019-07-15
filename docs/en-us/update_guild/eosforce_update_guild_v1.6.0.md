# v1.6.0 Operation steps of main network upgrading


### Upgrade content：

1. Merge EOSIO Version 1.7.4
2. Improve on-chain configuration function to facilitate subsequent updates
3. Partial code optimization





## 1. Node Upgrade (v1.6.0)

#### All nodes upgrade order: first complete the BP node upgrade, then upgrade the synchronous node

##### Preparation in advance
The configuration of this upgrade has changed. The config.ini file needs to delete the following options：

1. max-implicit-request
2. txn-reference-block-lag
3. reversible-blocks-db-size-mb
4. access-control-allow-credentials
5. connection-cleanup-period
6. network-version-match
7. sync-fetch-span
8. enable-stale-production
9. pause-on-startup
10. keosd-provider-timeout

 

### docker deployment

```
# docker name：eosforce-v1.6.0
docker pull eosforce/eos:v1.6.0
docker stop `original docker name`
docker run -d --name eosforce-v1.6.0 -v `local configuration directory`:/opt/eosio/bin/data-dir -v `local configuration directory`:/root/.local/share/eosio/nodeos -p 9876:9876 -p 8888:8888 eosforce/eos:v1.6.0 nodeosd.sh
# view the log
docker logs -f --tail 100 eosforce-v1.6.0
    
```
Verify upgrade results, version information：
```shell
docker exec -it eosforce-v1.6.0 opt/eosio/bin/cleos get info
"server_version_string": "force-v1.6.0"
```

### Source Code Compiling way
use tag: force-v1.6.0 


```shell
# Enter eosforce project directory

git fetch
git checkout force-v1.6.0
git submodule update --init --recursive
./eosio_build.sh
```

Notes (If compilation errors occur, you need to delete the opt directory in the HOME directory and recompile it)

Executable files produced under build/bin/ after compilation：cleos  keosd  nodeos (Start the service using nodeos only)

#### Files that need to be deployed in source code compilation way

```shell
configpath='~/eosforce/config' # your local service configuration directory

cp build/contracts/eosio.lock/eosio.lock.abi  build/contracts/eosio.lock/eosio.lock.wasm $configpath

cp build/contracts/System/System.abi build/contracts/System/System.wasm $configpath

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $configpath

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $configpath

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $configpath
```

#### Source Code Compiling way Start：
start nodeos

```shell
# start
nohup ./build/bin/nodeos --config-dir `configuration directory` --data-dir `data directory` > eos.log 2>&1 &

# view the log，Observe whether synchronization or block production is normal

tail -100f eos.log
```

#### after starting， verify upgrade result
version：


```shell
cleos -u http://127.0.0.1:8888 get info
"server_version_string": "force-v1.6.0"
```




## 2. Multi-Sign Update System Contracts

After the eosio.msig account initiates a multi-signature proposal, the node executes (it needs to use the command line to create a wallet to import the node account private key)：

```shell
# Approving multi-signature proposal for updating system contract code
cleos -u http://47.99.138.131:8888 multisig approve force.msig p.upsyscode '{"actor":"node account name","permission":"active"}' -p `node account name`@active

# Approving multi-signature proposal for updating system contract abi
cleos -u http://47.99.138.131:8888  multisig approve force.msig p.upsysabi '{"actor":"node account name","permission":"active"}' -p `node account name`@active
```
After mre than 2/3 nodes approve the prosal, we can execute the multi-signature update system contract.

```shell
# view proposal approvals
cleos  -u http://47.99.138.131:8888 get table eosio.msig force.msig approvals
```
