V1.4.1 mainnet update steps

This update adds candinate nodes sign-in function.

This function requires candinate nodes to set up heartbeat plugin, the plugin calls heartbeat system contract action periodically (default every ten minutes) to update the node state,
if the node has not updated the states for some time (default sixty minutes), the node will not receive the dividends.

1. Node update (v1.4.1) 

configuration file modification

Top twenty three BP nodes need not to modify the configuration file,
The candidate BP nodes need to modify config.ini file, add the following two options in the config.ini:

plugin = eosio::heartbeat_plugin

bp-mapping=biosbpa=KEY:biosbpaa

Note: The candidate needs to create an account using the node's signing key's corresponging public key, for example biosbpaa, and the newly created account needs to have sufficient available balance to pay for the heartbeat action fee.

Other configuration files keep unchanged:
ctiveacc.json: https://updatewallet.oss-cn-hangzhou.aliyuncs.com/eosforce/activeacc.json (md5sum b7c295454e6e81ab3023baeebf2d9131)
genesis.json: keeps unchanged

docker deployment

# container name£ºeosforce-v1.4.1
docker pull eosforce/eos:v1.4.1
docker stop container name
docker run -d --name eosforce-v1.4.1 -v local configuration directory:/opt/eosio/bin/data-dir -v local data directory:/root/.local/share/eosio/nodeos -p 9876:9876 -p 8888:8888 eosforce/eos:v1.4.1 nodeosd.sh
# view the log
docker logs -f --tail 100 eosforce-v1.4.1

Verify the update results, version information:

docker exec -it eosforce-v1.4.1 opt/eosio/bin/cleos get info
"server_version_string": "force-v1.4.1"

in source compling way:

fetch tag: force-v1.4.1

# enter eosforce project directory
git fetch
git checkout force-v1.4.1
git submodule update --init --recursive
./eosio_build.sh

After compling, use the generated executable files in build/bin/ directory: cleos keosd nodeos

(starting service needs only nodeosd.)

Set up the configuration

onfigpath='~/eosforce/config' #your configuration directory

cp build/contracts/eosio.lock/eosio.lock.abi  build/contracts/eosio.lock/eosio.lock.wasm $configpath

cp build/contracts/System/System.abi build/contracts/System/System.wasm $configpath

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $configpath

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $configpath

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $configpath

add the two options in the config.ini explained in previous steps:
plugin = eosio::heartbeat_plugin

bp-mapping=biosbpa=KEY:biosbpaa

starting:

# starting the nodeosd
nohup ./build/bin/nodeos --config-dir configuration directory --data-dir data directory > eos.log 2>&1 &

# view the log£¬watch if syncing or block production function well 
tail -100f eos.log

Verify update results

cleos -u http://127.0.0.1:8888 get info
"server_version_string": "force-v1.4.1"

2. update system contract by multisig

After force.msig initiates multisig proposal, the BP node execute the following command: (Need to import BP account private key into wallets)

# approve update system contract code multisig proposal
cleos  -u https://w1.eosforce.cn multisig approve force.msig p.upsyscode '{"actor":"node account name","permission":"active"}' -p node account name@active
# approve update system contract abi multisig proposal
cleos  -u https://w1.eosforce.cn multisig approve force.msig p.upsysabi '{"actor":"node account name","permission":"active"}' -p node account name@active

The proposal can be executed after more than 2/3 BP nodes approved it.

# view the proposal approval information
cleos  -u https://w1.eosforce.cn get table eosio.msig force.msig approvals
