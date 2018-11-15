# EosForce bp deploy

--------------------------

!> Important Notes:

- The minimum server requirements is dual-core CPU, 4G RAM, 50G SSD hardrive, and the recommended system is 64-bit ubuntu 16.04
- EOSForce mainnet cannot be deployed on the same server as the EOS EMLG mainnet, meaning that one server can only deploy one mainnet. This is to prevent unusual errors during deployment
- Third-party plug-ins on the EOSForce ecosystem are not compatible with the EOS EMLG Mainnet. For example, EMLG eosjs plug-ins cannot dock the EOSForce mainnet
- Before the deployment of EOSForce node, please get familar with EOS-related documents, the use of command client and RPC API use for example
- For BP node deployment, modify the settings based on deploying syncronous node, and then it will be a BP node
- For BP registraton, please have 100 EOSC in the wallet account in advance for registration fees.

## sync node deploy

This is the EOSForce source code deployment plan based on linux ubuntu 16.04. Refer to docker deployment here: https://github.com/eosforce/genesis

### 1. Download the srouce

```bash
apt-get update && apt-get install -y git wget
git clone https://github.com/eosforce/eosforce.git eosforce
```

### 2. Excute the following commands to instal EOSForce

```bash
cd eosforce && git submodule update --init --recursive && ./eosio_build.sh
mkdir -p ~/.local/share/eosio/nodeos/config
curl https://raw.githubusercontent.com/eosforce/genesis/master/genesis.json -o ~/.local/share/eosio/nodeos/config/genesis.json
cp build/contracts/System/System.abi build/contracts/System/System.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/eosio.bios/eosio.bios.abi build/contracts/eosio.bios/eosio.bios.wasm ~/.local/share/eosio/nodeos/config
cd build && make install
```

### 3. Obtain and modify the config core setting documents

```bash
wget http://download.aitimeout.site/config.ini
cp config.ini ~/.local/share/eosio/nodeos/config/
```
Modify the config.ini document in 2 places:

(1) p2p-server-address = ip:7894 (ip is the public server IP. Firewall must release the port, and the port modifies itself)

(2) modify to your own genesis.json path. Use absolute path to prevent errors


### 4. startup

```bash
cd build/programs/nodeos && ./nodeos
```

#### get info

```bash
cleos get info
```

explore the chain info on major eosforce network

https://w1.eosforce.cn/v1/chain/get_info 


## BP Node Deployment

### Preparation

Generate a public/private key pair for BP node. Execute the command as follows:


```bash
cleos create key
```

Execution result as follows:

```bash
Private key: 5KidVdxbLKbJo9QiTyrbYULNTdKFTzdCb9oZgdaWye2CZfXz2hC
Public key: EOS6Z4fD6isTKZwaeH6Req7QXZLK3Yvb2rQoTxefVcsGXaXsFrBap
```

Public key will be used during BP registration later, that is, updatebp.

Modify 2 places based on the already deployed syncronous node:

config.ini:

(1) producer-name = bpname (bpname being the name you have as a BP)

(2) signature-provider = EOSpubkey = KEY:EOSprivkey (EOSpubkey & EOSprivkey being the public/private keys generated earlier)

### Launch node, execute the following commands:

Launch:

```bash
ps -aux|grep nodeos
kill -2 'nodeos pid'
cd build/programs/nodeos && ./nodeos
```

## BP Node Registration

Preparation: first register a EOSForce account name and transfer 100 EOSC to this account. Make sure the account name is the same as bpname. Right now you should have pub_key, pri_key, and bpname.

Create a wallet:

```bash
cleos wallet create
```

Results as follows:
  
```bash
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5HwhcFEfN2Up63iK5LfQwXX7FmkUNkwV1t4TG73tMNxj59YeQws"
```

Default wallet. The last line being the wallet password.

Import account private key to the wallet:

```bash
cleos wallet import pri_key
```

Execute command to register:

```bash
cleos -u https://p1.eosforce.cn push action eosio updatebp '{"bpname":"bpname","block_signing_key":"block_signing_key","commission_rate":"commission_rate","url":"https://eosforce.io"}' -p bpname
```

Successful registration returns the following result:

```bash
executed transaction: 34dbe8bb08d0f7c3d5a4453d1e068e35f03c96f25d200c4e2a795e6aec472d60  160 bytes  6782 us
#         eosio <= eosio::transfer              {"from":"eosforce","to":"user1","quantity":"10.0000 EOS","memo":"my first transfer"}
warning: transaction executed locally, but may not be confirmed by the network yet
```

bpname being the name of the BP. block_signing_key being BP’s public key. commission_rate being the amount of BP reward allocated to the BP itself(e.g. 3000 would be giving 70% of the reward back to the voters and 30% to BP itself). Refer to the details at: 
https://github.com/eosforce/contracts/tree/master/System


### How to verify block production

2 conditions for block production:

- Top 23 BPs can produce blocks
- Local BP node syncronization is complete

Download EOSForce local wallet to check if your BP is producing blocks: 
https://github.com/eosforce/wallet-desktop/releases
