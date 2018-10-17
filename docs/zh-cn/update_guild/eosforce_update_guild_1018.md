# 主网升级操作文档 2018-10-18

原力主网计划在10月18日下午升级到v1.1.0 master版本，下面是操作指南。

这次升级主要包含三方面内容：

- 1. 合约开放临时方案
- 2. 新的分红方案实现
- 3. 合并EOS更新

这次升级需要使用 master分支上的 v1.1.0 版本，需要更新配置并且添加System01系统合约。

首先config.ini配置文件中需要添加以下配置：

```
http-validate-host=false
```

其次，需要新的分红系统合约：

```
build/contracts/System01/System01.abi
build/contracts/System01/System01.wasm
```

下面依照部署方式分别叙述：

## 1. 编译安装方式

### 编译

```
git clone git@github.com:eosforce/eosforce.git eosforce-build

cd eosforce-build

git submodule update --init --recursive

./eosio_build.sh

rm -rf ./eosforce
mkdir eosforce
mkdir eosforce/config

cd eosforce

cp ../build/contracts/eosio.bios/eosio.bios.abi ./config 
cp ../build/contracts/eosio.bios/eosio.bios.wasm ./config
cp ../build/contracts/eosio.msig/eosio.msig.abi ./config 
cp ../build/contracts/eosio.msig/eosio.msig.wasm ./config
cp ../build/contracts/eosio.token/eosio.token.abi ./config
cp ../build/contracts/eosio.token/eosio.token.wasm ./config
cp ../build/contracts/System/System.abi ./config 
cp ../build/contracts/System/System.wasm ./config

cp ../build/contracts/System01/System01.abi ./config 
cp ../build/contracts/System01/System01.wasm ./config

cp ../build/programs/cleos/cleos ./
cp ../build/programs/nodeos/nodeos ./ 
cp ../build/programs/keosd/keosd ./ 

tar -cvf eosforce-build.tar ./*

cp ./eosforce-build.tar ~/
```

以上命令构建了一个`eosforce-build.tar`包含所需文件，当然运维也可以选择其他方式打包，总之我们需要以下文件：

- nodeos 
- System01.wasm
- System01.abi
- eosio.bios.abi
- eosio.bios.wasm
- eosio.msig.abi
- eosio.msig.wasm
- eosio.token.abi
- eosio.token.wasm
- System.abi
- System.wasm

其中包括了之前的系统合约，如果是升级的话，因为之前的系统合约已经有了，所以只需要以下文件：

- nodeos 
- System01.wasm
- System01.abi

### 更新

下面是更新的步骤，更新之前需要确认之前运行的nodeos的数据信息路径和配置信息路径，
首先使用2信号量向nodeos发送退出命令，保证数据文件不被损坏。

```
kill -2 {nodeos pid}
```

之后备份数据文件和配置文件，将`System01.wasm`和`System01.abi`复制进配置文件目录下，
注意此时该目录下还有其他系统合约配置：

```
cp ./System01.wasm /path/to/nodeos/config/
cp ./System01.abi /path/to/nodeos/config/
```

注意：一般配置中会将配置文件路径设置为 ~/.local/share/eosio/nodeos/config
之前应该有以下文件：

- config.ini
- eosio.bios.abi
- eosio.bios.wasm
- eosio.msig.abi
- eosio.msig.wasm
- eosio.token.abi
- eosio.token.wasm
- System.abi
- System.wasm

修改config.ini文件，加入以下配置：

```
http-validate-host=false
```

最后启动升级后的服务

```
cd build/programs/nodeos &&  nohup ./nodeos >> nodeos.log 2>&1 &
```


## 2. 使用docker

新的版本是 eosforce/eos:v1.1.0

需要拉取镜像：

```
 docker pull eosforce/eos:v1.1.0
```

先关闭老的服务，并备份数据文件

```
  docker stop eosforce
```

升级只需要用新景象启动即可，注意修改以下参数中的配置为之前的配置。

```
  docker run -d --restart=always --name eosforce-new -v /data/eosforce:/opt/eosio/bin/data-dir -v /data/nodeos/eosforce:/root/.local/share/eosio/nodeos -p 8888:8888 -p 9876:9876 eosforce/eos:v1.1 nodeosd.sh
```


## 3. 注意事项

在启动新的服务前一定要备份所有文件。