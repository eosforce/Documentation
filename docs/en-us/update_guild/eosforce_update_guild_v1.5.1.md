
V1.5.1 EOSForce Mainnet Update Operating Steps


The upgrade includes:
1. Merge EOSIO code updates
2. BP punishment mechanism
3. Improvement of fee scheme
4. Switch voting with no redemption period


Block data package download address of Docker deployment scheme:https://updatewallet.oss-cn-hangzhou.aliyuncs.com/dokcer-nodeos-latest.tar.gz


1. Node upgrade (v1.5.1)

Nodes upgrade sequence: upgrade the Block Producers first, then upgrade the synchronization nodes


Docker deployment


# Container name: eosforce-v1.5.1
docker pull eosforce/eos:v1.5.1
Docker stop original-container-name
Docker run -d --name eosforce-v1.5.1 -v Local-config-directory: /opt/eosio/bin/data-dir -v Local-data-directory: /root/.local/share/eosio/nodeos -p 9876:9876 -p 8888:8888 eosforce/eos:v1.5.0 nodeosd.sh

# Check the log
docker logs -f --tail 100 eosforce-v1.5.1
    
Verify upgrade results and the version information:

docker exec -it eosforce-v1.5.1 opt/eosio/bin/cleos get info
"server_version_string": "force-v1.5.1"

Source code compilation

Use tag: force-v1.5.1

# Enter the eosforce project directory

git fetch
git checkout force-v1.5.1
git submodule update --init --recursive
./eosio_build.sh


After compiling, use the executable file generated under build/bin/: cleos keosd nodeos



(Start the service only using nodeos)

Source code compilation, files that need to be configured


Configpath='~/eosforce/config' #Modify the configpath to the local service config directory

cp build/contracts/eosio.lock/eosio.lock.abi  build/contracts/eosio.lock/eosio.lock.wasm $configpath

cp build/contracts/System/System.abi build/contracts/System/System.wasm $configpath

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $configpath

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $configpath

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $configpath


Source compilation mode - start:

Start the nodeos


# Start


nohup ./build/bin/nodeos --config-dir config-directory --data-dir data-directory > eos.log 2>&1 &


# Check the logs to see if synchronization or block-generating is working

tail -100f eos.log


After the compilation mode starts, verify the upgrade results
Version information:

cleos -u http://127.0.0.1:8888 get info
"server_version_string": "force-v1.5.1"



2. Update system contract with multi-sign
After EOSForce force.msig account initiates the multi-sign proposal, the nodes executes it(it needs to use the command line to create the wallet and import the private key of the node account) :



# Approve the updating system contract code multi-sign proposal


cleos  -u https://w1.eosforce.cn multisig approve force.msig p.upsyscode '{"actor":"node-account-name","permission":"active"}' -p node-account-name@active


# Approve the updating system contract abi multi-sign proposal



cleos  -u https://w1.eosforce.cn multisig approve force.msig p.upsysabi '{"actor":"node-account-name","permission":"active"}' -p node-account-name@active



If more than 2/3 of the nodes have successfully executed, the multi-sign updating system contracts can be efficient.


# Check the proposal approval status
cleos  -u https://w1.eosforce.cn get table eosio.msig force.msig approvals

