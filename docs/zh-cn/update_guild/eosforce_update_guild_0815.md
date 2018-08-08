# 2018年8月15日BP节点预演说明

因EOSFORCE要上线新版本，提供更加可靠健壮，功能丰富的eosforce主网，本次预演由当前竞选BP节点社区共同完成一次同一个主网的升级预演，需要各BP节点社区配合原力这边的技术支持，更新自己BP节点服务，即停服务，替换成由原力提供最新代码编译出来的可执行程序来启动主网

## 事项和规则
1. 旧版本eosforce主网和当前线上eosforce主网一致，新版本eosforce主网是原力官方即将发布的eosforce主网

2. 本次主网升级预演完全在测试环境进行，切不可动已运行在线上的eosforce主网节点

3. 升级由BP节点一个一个按固定顺序（23个BP节点按节点投票排名由高到低的顺序来完成升级），前一BP升级成功，下一个BP节点开始完成升级，确保主网最后预演成功


## 预演流程

本次预演升级是基于测试环境进行的，为了后面更顺利更安全的完成线上eosforce主网系统的升级，本次预演需要每个BP节点完成以下7个步骤：

1. 准备工作： 这次节点升级预演基于在原力eos主网测试环境下进行，各bp社区的需要准备一台测试机器（配置为ubuntu 16.04 64位linux 操作系统，配置2核4G内存以上）作为这次预演的测试机器
 
2.  旧版本eosforce主网部署： 旧版本bp节点和当前线上eosforce代码一样， 需要各社区在搭建一套测试环境模拟真实的线上，具体是现役23个超级节点在已准备好的测试机器上基于旧版本的eosforce的代码来重新部署一次bp节点，这样旧版本bp节点完成部署（需完成节点部署，bp节点注册）

3. 旧版本eosforce主网测试： 基于部署好的后各bp节点开始进行出块，投票等其他相关测试

4. 停止旧版本eosforce主网服务： 测试如果通过，停止旧版本的nodeos进程服务并进行下一步，否则退回上一步

5. 新版本eosforce主网的升级：基于上一步已停掉的旧版本的eosforce主网来完成升级，具体是由我们提供的最新eosforce的代码来编译生成的可执行文件替换旧版本的可执行文件，启动服务完成升级（需完成节点部署，bp节点注册）

6. 新版本eosforce主网测试： 各bp节点开始进行 出块，投票等其他相关测试

7. 预演结束： 测试如果通过这次预演成功结束，否则退回上一步

## 新版本eosforce主网部署说明

我们会提供2种部署方式： 源码编译部署和docker镜像部署
下面是源码部署方式，其中docker部署方式后续在文档中更新

### 1. 下载源码

```shell
apt-get update && apt-get install -y git wget
git clone https://github.com/eosforce/eosforce.git eosforce
git checkout -b release 
```

### 2. 执行如下命令安装原力eos

```shell
cd eosforce && git submodule update --init --recursive && ./eosio_build.sh
mkdir -p ~/.local/share/eosio/nodeos/config
curl https://raw.githubusercontent.com/eosforce/genesis/master/genesis.json -o ~/.local/share/eosio/nodeos/config/genesis.json
cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/System/System.abi build/contracts/System/System.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/eosio.bios/eosio.bios.abi build/contracts/eosio.bios/eosio.bios.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm ~/.local/share/eosio/nodeos/config
cp config.ini ~/.local/share/eosio/nodeos/config/
cd build && make install
```

### 3. config配置文件修改

#### 生成公私钥

生成一对公私钥给BP节点使用，执行如下命令生成


```shell
cleos create key
```

执行结果如下：

	Private key: 5KidVdxbLKbJo9QiTyrbYULNTdKFTzdCb9oZgdaWye2CZfXz2hC
	Public key: EOS6Z4fD6isTKZwaeH6Req7QXZLK3Yvb2rQoTxefVcsGXaXsFrBap

其中 Private key为私钥 Public key为公钥,   注意Public key在后面注册bp时用到，即执行updatebp时使用

config.ini文件需要修改4个地方：

第1个修改地方：p2p-server-address = ip:9876 (ip为公网服务器ip，端口自行修改，注意防火墙要放行该端口)

第2个修改的地方，用绝对路径防止出错：

genesis-json = "/root/.local/share/eosio/nodeos/config/genesis.json"

第3需要修改的地方：

producer-name = bpname （bpname为你的bp的名称）

第4个修改地方：

signature-provider = EOSpubkey=KEY:EOSprivkey （其中EOSpubkey准备工作中生成的公钥，EOSprivkey为准备工作中生成的私钥）

第5个修改地方：

p2p-peer-address = 7894 (这个配置需要修改) 

### 4.启动

	cd build/programs/nodeos && ./nodeos
	
	
## BP节点注册

准备工作：
首先要注册一个原力eos账户名，账户名必须和bp的名字一样，也为bpname，这样
就有了公钥pub_key,私钥pri_key,账户名 bpname（和bp的名字同名），这个步骤就不详细介绍

创建一个钱包

	cleos wallet create

  结果如下：
  
  	Creating wallet: default
	Save password to use in the future to unlock this wallet.
	Without password imported keys will not be retrievable.
	"PW5HwhcFEfN2Up63iK5LfQwXX7FmkUNkwV1t4TG73tMNxj59YeQws"
	
  生成默认的钱包，最后一行为钱包的密码
  
导入账户私钥到钱包

	cleos wallet import pri_key

执行命令进行注册 (注意-u 指定为我们线上搭建的一台测试节点的bp节点，切不可指定正式环境的地址来注册)

	cleos -u http://47.98.249.86:8888 push action eosio updatebp '{"bpname":"bpname","block_signing_key":"block_signing_key","commission_rate":"commission_rate","url":"https://eosforce.io"}' -p bpname

注册成功返回如下结果：

	executed transaction: 34dbe8bb08d0f7c3d5a4453d1e068e35f03c96f25d200c4e2a795e6aec472d60  160 bytes  6782 us
	#         eosio <= eosio::transfer              {"from":"eosforce","to":"user1","quantity":"10.0000 EOS","memo":"my first transfer"}
	warning: transaction executed locally, but may not be confirmed by the network yet


其中bpname 为bp的名称。block_signing_key为BP的公钥，commission_rate为佣金比例，设置3000，就是给用户分红70%，bp为30%，详细介绍可以参考 https://github.com/eosforce/contracts/tree/master/System


### 最后如何检车出块

#### 出块的2个条件

* 投票后排名前23名，可出块
* 本地bp节点同步已完成
