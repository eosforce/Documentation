# EosForce bp deploy

--------------------------

!> note

- 一般服务器最低配置为2核cpu4G内存 50G ssd硬盘,系统推荐64位ubuntu 16.04操作系统
- 部署过程中原力eos主网不可和联盟eos主网同时部署在一台服务器上，即一台服务器只能部署一套EOS主网，主要防止使用过程中出现奇怪的错误
- 原力eos生态的第三方应用插件不可和联盟eos主网混合使用，比如联盟的eosjs插件不可和原力eos主网直接对接
- 部署原力eos节点前，最好需要之前有过eos的相关的基础知识学习，比如命令行客户端及RPC API使用
- 部署BP节点，是部署同步节点的基础上修改下配置，就变成BP节点 
- 注册bp时，需要钱包账户最低充值100个eos作为注册费

## sync node deploy

基于linux操作系统 ubuntu 16.04版本 原力eos源码部署方案，docker deploy reference:  https://github.com/eosforce/genesis

### 1. download the srouce

```bash
apt-get update && apt-get install -y git wget
git clone https://github.com/eosforce/eosforce.git eosforce
```

### 2. exe shell to build eos

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

### 3. config

```bash
wget http://download.aitimeout.site/config.ini
cp config.ini ~/.local/share/eosio/nodeos/config/
```

config.ini update:

p2p-server-address = ip:port
genesis-json = "/root/.local/share/eosio/nodeos/config/genesis.json"


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


## BP节点部署

### 准备工作

生成一对公私钥给BP节点使用，执行如下命令生成

```bash
cleos create key
```

执行结果如下：

```bash
Private key: 5KidVdxbLKbJo9QiTyrbYULNTdKFTzdCb9oZgdaWye2CZfXz2hC
Public key: EOS6Z4fD6isTKZwaeH6Req7QXZLK3Yvb2rQoTxefVcsGXaXsFrBap
```

其中 Private key为私钥 Public key为公钥, Private  注意Public key在后面注册bp时用到，即执行updatebp

**基于以上部署好的同步节点进行修改，只需修改2个地方:**

config.ini修改如下：

第一修改的地方：

producer-name = bpname （bpname为你的bp的名称）

第二修改地方：

signature-provider = EOSpubkey=KEY:EOSprivkey （其中EOSpubkey准备工作中生成的公钥，EOSprivkey为准备工作中生成的私钥）

### 启动节点,执行如下命令

删除旧的数据

```bash
rm -rf ~/.local/share/eosio/nodeos/data
```

启动

```bash
cd build/programs/nodeos && ./nodeos
```

## BP节点注册

准备工作：
首先要注册一个原力eos账户名，并需要给这个账户转100个eos，注册时需要注册费，账户名必须和bp的名字一样，也为bpname，这样
就有了公钥pub_key,私钥pri_key,账户名 bpname（和bp的名字同名），这个步骤就不详细介绍

创建一个钱包

```bash
cleos wallet create
```

结果如下：
  
```bash
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5HwhcFEfN2Up63iK5LfQwXX7FmkUNkwV1t4TG73tMNxj59YeQws"
```

生成默认的钱包，最后一行为钱包的密码
  
导入账户私钥到钱包

```bash
cleos wallet import pri_key
```

执行命令进行注册

```bash
cleos -u https://p1.eosforce.cn push action eosio updatebp '{"bpname":"bpname","block_signing_key":"block_signing_key","commission_rate":"commission_rate","url":"https://eosforce.io"}' -p bpname
```

注册成功返回如下结果：

```bash
executed transaction: 34dbe8bb08d0f7c3d5a4453d1e068e35f03c96f25d200c4e2a795e6aec472d60  160 bytes  6782 us
#         eosio <= eosio::transfer              {"from":"eosforce","to":"user1","quantity":"10.0000 EOS","memo":"my first transfer"}
warning: transaction executed locally, but may not be confirmed by the network yet
```

其中bpname 为bp的名称。block_signing_key为BP的公钥，commission_rate为佣金比例，设置3000，就是给用户分红70%，bp为30%，详细介绍可以参考 https://github.com/eosforce/contracts/tree/master/System


### 最后如何检验出块

出块的2个条件:

* 投票后排名前23名，可出块
* 本地bp节点同步已完成

可以下载我们的eosforce钱包并安装，查看自己的BP有没有出块
(钱包下载地址：https://github.com/eosforce/wallet-desktop/releases)
