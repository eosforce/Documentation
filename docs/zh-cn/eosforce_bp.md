# EOSForce 节点部署指南

--------------------------

!> 注意事项

- 一般服务器最低配置为 **4核cpu** **8G内存** **120G ssd硬盘**,系统推荐64位ubuntu **18.04**操作系统
- 部署过程中原力eos主网不可和EMLG EOS主网同时部署在一台服务器上，即一台服务器只能部署一套EOS主网，主要防止使用过程中出现奇怪的错误
- 原力eos生态的第三方应用插件不可和EMLG EOS主网混合使用，比如EMLG EOS的eosjs插件不可和原力eos主网直接对接
- 部署原力eos节点前，最好需要之前有过eos的相关的基础知识学习，比如命令行客户端及RPC API使用
- 部署BP节点，是部署同步节点的基础上修改下配置，就变成BP节点
- 注册bp时，需要钱包账户最低充值100个eos作为注册费

## 1. 基于Docker镜像部署同步节点

基于Docker部署镜像是一种十分方便安全的方式, 节点可以通过docker hub直接获取镜像, 也可以基于源码自行构建镜像.

### 1.1 获取镜像

节点可以通过docker hub直接获取镜像:

```bash
docker pull eosforce/node:1.7.0
```

### 1.1 构建镜像

目前EOSForce Docker支持还在完善中, 所以没有直接整合到eosforce项目之中, 目前位于[eosforce-docker](https://github.com/fanyang1988/eosforce-docker)

考虑到某些地区的网络环境, 这里采取的方式是基于已经在ubuntu 18.04系统下编译完毕的eosforce二进制文件构建镜像,
编译方法可以根据下面第二节的叙述, 这里假设用户按照第二节中默认的项目源码位置($HOME/eosc)来进行编译.

```bash
git clone git@github.com:fanyang1988/eosforce-docker.git
cd eosforce-docker
export EOSFORCE_ROOT=$HOME/eosc
cd scripts
./build-fast-docker.sh
```

脚本编译出的镜像默认名字为`eosforce/node:1.7.0`

### 1.2 基于镜像启动节点

首先需要建立好容器数据目录:

```bash
mkdir $HOME/eosc-node
mkdir -p $HOME/eosc-node/config
mkdir -p $HOME/eosc-node/data
```

这里我们使用`$HOME/eosc-node`作为容器数据的存储位置, 如果之前运行过节点, 则可以在安全关闭节点之后将其数据文件复制过来, 使得目录文件如下:

```base
tree
.
└── eosc-node
    ├── config
    │   ├── activeacc.json
    │   ├── config.ini
    │   ├── eosio.lock.abi
    │   ├── eosio.lock.wasm
    │   ├── eosio.msig.abi
    │   ├── eosio.msig.wasm
    │   ├── eosio.token.abi
    │   ├── eosio.token.wasm
    │   ├── genesis.json
    │   ├── System01.abi
    │   ├── System01.wasm
    │   ├── System02.abi
    │   ├── System02.wasm
    │   ├── System.abi
    │   └── System.wasm
    └── data
        ├── blocks
        │   ├── blocks.index
        │   ├── blocks.log
        │   └── reversible
        │       ├── shared_memory.bin
        │       └── shared_memory.meta
        ├── snapshots
        └── state
            ├── shared_memory.bin
            └── shared_memory.meta
```

如果是新启动节点则docker镜像会补充必要的文件, 数据需要重新同步, 在启动之后需要修改对应的config.ini(在$HOME/eosc-node/config目录下)中的p2p节点配置,
默认配置中为原力团队提供的根节点, 这些节点目前连接数较多, 节点需要联系其他超级节点和同步节点与其互联来提高网络稳定性.

p2p配置在config.ini中:

```bash
p2p-peer-address = 47.99.138.131:9876
p2p-peer-address = 47.99.165.99:9876
p2p-peer-address = 47.99.167.137:9876
p2p-peer-address = 47.99.151.178:9874
...
```

添加即可.

这时可以启动节点:

```bash
docker run -d —name eosc-node -v $HOME/eosc-node:/eosforce -p 8888:8888 -p 9876:9876 eosforce/node:1.7.0 nodeosd.sh
```

!> 注意 和老版本对比启动最大的区别是不需要单独为config和data指定不同的volume

可以通过docker的stop, start, restart命令停止, 启动, 重启节点

关闭节点以修改配置:

```bash
docker stop eosc-node
```

重启节点:

```bash
docker start eosc-node
```

!> 注意, 节点进程需要发送信号以安全退出, 如果强制退出则会造成数据损坏, 需要重跑区块.

## 2. 基于源码编译部署同步节点

!> 注意: **不要**使用root账户部署编译EOSForce

### 2.1. 下载源码

```bash
apt-get update && apt-get install -y git wget
git clone https://github.com/eosforce/eosforce.git $HOME/eosc
```

### 2.2. 执行如下命令安装原力

```bash
cd $HOME/eosc && git fetch && git checkout force-v1.7.1 && git pull && git submodule update --init --recursive
cd ./scripts/ && ./eosio_build.sh -y
```

如果环境中第一次编译eosforce, 需要sudo来安装部分依赖包.

!> 注意: EOSForce不建议将其安装到系统路径下, 如果之前系统安装过eosio或者eosforce, 编译将会失败, 需要手动删去`/usr/local/eosio`目录

!> 注意: 编译过程中部分中间文件将会存储在`$HOME/eosforce`下, 所以最好不要将代码clone在`$HOME`下.

编译成功后二进制文件在代码目录下的`build/bin`下.

### 2.3. config核心配置文件获取并修改(若想修改p2p地址请参考第二节)

EOSForce节点启动时需要配置文件和创世合约数据, 前者需要用户配置而后者为固定的数据, 只需放入config目录下.

首先我们建立config目录:

```bash
mkdir -p $HOME/eosforce-node/config
cd $HOME/eosforce-node/config
wget https://updatewallet.oss-cn-hangzhou.aliyuncs.com/eosforce/genesis/genesis-contracts.tar.gz
tar -xzvf ./genesis-contracts.tar.gz
wget https://updatewallet.oss-cn-hangzhou.aliyuncs.com/eosforce/genesis/genesis-datas.tar.gz
tar -xzvf ./genesis-datas.tar.gz
wget https://updatewallet.oss-cn-hangzhou.aliyuncs.com/eosforce/genesis/config-template.tar.gz
tar -xzvf ./config-template.tar.gz
```

执行完后config下的config.ini文件需要修改：

p2p-server-address = ip:7894 (ip为公网服务器ip，端口自行修改，注意防火墙要放行该端口)

### 2.4. 启动节点并测试

```bash
cd $HOME/eosc/build/bin/
./nodeos -d $HOME/eosforce-node/data --config-dir $HOME/eosforce-node/config 
```

#### 打开另一个终端查看本地区块高度及对比eos原力官方主网的出块高度

查看本地高度命令如下，并多次执行如下命令区块高度为不断增加，说明同步正常，直到高度和原力主网高度接近时，同步完成

```bash
$HOME/eosc/build/bin/cleos -u http://127.0.0.1:8888 get info
```

查看原力eos主网区块高度

```bash
curl --request POST \
  --url https://w1.eosforce.cn:443/v1/chain/get_info \
  --header 'accept: application/json' \
  --header 'content-type: application/json' \
  --data '{}'
{"server_version":"24d56e6b","chain_id":"bd61ae3a031e8ef2f97ee3b0e62776d6d30d4833c8f7c1645c657b149151004b","head_block_num":12133019,"last_irreversible_block_num":12132988,"last_irreversible_block_id":"00b9227c03303b1e141b8b8aa6e5556797bcc2e40b90be731381f3b893684a85","head_block_id":"00b9229be7111bb6dcc2a9068131354adf0539ff7a97df1cd9f2a629b437a571","head_block_time":"2019-08-27T13:52:42.000","head_block_producer":"eostrust","virtual_block_cpu_limit":1000000000,"virtual_block_net_limit":1048576000,"block_cpu_limit":999900,"block_net_limit":1048576,"server_version_string":"force-v1.6.0"}
```

其中head_block_num为区块高度.

## 3. BP节点部署

### 3.1 准备工作

生成一对公私钥给BP节点使用，执行如下命令生成

```bash
$HOME/eosc/build/bin/cleos create key
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

类似与下面:

```bash
producer-name = bpname
signature-provider = EOS6Z4fD6isTKZwaeH6Req7QXZLK3Yvb2rQoTxefVcsGXaXsFrBap=KEY:5KidVdxbLKbJo9QiTyrbYULNTdKFTzdCb9oZgdaWye2CZfXz2hC
```

### 3.2 启动节点,执行如下命令

启动

```bash
ps -aux|grep nodeos
kill -2 'nodeos pid'
cd build/programs/nodeos && ./nodeos
```

## 4. BP节点注册

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
cleos -u https://w1.eosforce.cn push action eosio updatebp '{"bpname":"bpname","block_signing_key":"block_signing_key","commission_rate":"commission_rate","url":"https://eosforce.io"}' -p bpname
```

注册成功返回如下结果：

```bash
executed transaction: 34dbe8bb08d0f7c3d5a4453d1e068e35f03c96f25d200c4e2a795e6aec472d60  160 bytes  6782 us
#         eosio <= eosio::transfer              {"from":"eosforce","to":"user1","quantity":"10.0000 EOS","memo":"my first transfer"}
warning: transaction executed locally, but may not be confirmed by the network yet
```

其中bpname 为bp的名称。block_signing_key为BP的公钥，commission_rate为佣金比例，设置3000，就是给用户分红70%，bp为30%，详细介绍可以参考 https://github.com/eosforce/contracts/tree/master/System

## 5. 最后如何检验出块

出块的2个条件:

- 投票后排名前23名，可出块
- 本地bp节点同步已完成

可以下载我们的eosforce钱包并安装，查看自己的BP有没有出块
(钱包下载地址：https://github.com/eosforce/wallet-desktop/releases)
