# v1.5.2 Operation steps of main network updating


### Upgrade content：

1. vote age deduction Fee
2. Optimizing System Contract Implementation





## 1. Node upgrade (v1.5.2)

#### All nodes upgrade order, first complete the BP node upgrade, then upgrade the synchronous node


 

### docker deployment

```
# docker container name：eosforce-v1.5.2
docker pull eosforce/eos:v1.5.2
docker stop original docker container name
docker run -d --name eosforce-v1.5.2 -v local configuration directory:/opt/eosio/bin/data-dir -v local data directory:/root/.local/share/eosio/nodeos -p 9876:9876 -p 8888:8888 eosforce/eos:v1.5.2 nodeosd.sh
# view log
docker logs -f --tail 100 eosforce-v1.5.2
    
```
verify upgrade result, version information：
```shell
docker exec -it eosforce-v1.5.2 opt/eosio/bin/cleos get info
"server_version_string": "force-v1.5.2"
```

### source compiling 
use tag: force-v1.5.2

```shell
# enter eosforce project directory
git fetch
git checkout force-v1.5.2
git submodule update --init --recursive
./eosio_build.sh
```

Executable files produced under build/bin/ after compilation：cleos  keosd  nodeos

(Start the service using nodeos only)

#### Files that need to be configured for source code compilation
```shell
configpath='~/eosforce/config' # Modify the configuration directory for your local service

cp build/contracts/eosio.lock/eosio.lock.abi  build/contracts/eosio.lock/eosio.lock.wasm $configpath

cp build/contracts/System/System.abi build/contracts/System/System.wasm $configpath

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $configpath

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $configpath

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $configpath
```

#### Source Code Compiling Mode Start：
Start nodeos

```shell
# Start
nohup ./build/bin/nodeos --config-dir configuration directory --data-dir data directory > eos.log 2>&1 &

# View logs to see if synchronization or blocking is normal

tail -100f eos.log
```

#### Verify the upgrade results after the compilation mode starts
version information：


```shell
cleos -u http://127.0.0.1:8888 get info
"server_version_string": "force-v1.5.2"
```


## 2. Multi-Sign Update System Contracts

After the eosio.msig account initiates a multi-signature proposal, the node executes (it needs to use the command line to create a wallet to import the node account's private key)

```shell
# Approving multi-signature proposal for updating system contract code
cleos  -u https://w1.eosforce.cn multisig approve eosio.msig p.upsyscode '{"actor":"node account name","permission":"active"}' -p node account name@active
# Approving multi-signature proposal for updating system contract abi
cleos  -u https://w1.eosforce.cn multisig approve eosio.msig p.upsysabi '{"actor":"node account name","permission":"active"}' -p node account name@active
```

After more than 2/3 nodes approve, we can execute the multi-signature update system contract.

```shell
# Check the approval of the proposal
cleos  -u https://w1.eosforce.cn get table eosio.msig eosio.msig approvals
```