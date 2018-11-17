# eosforce v1.2.0.test 主网升级预演说明

docker版本启动 与 源码编译启动方式取一即可，注意两种版本启动节点产生的data文件互不兼容。

-----
## docker 版本：

### 1. 拉取最新版本镜像

```shell
docker pull eosforce/eos:v1.2.0.test
```

### 2. 配置文件 config.ini

配置p2p地址，端口、bp等

```shell
p2p-peer-address = 47.99.138.131:9076
p2p-peer-address = 47.99.165.99:9076
p2p-peer-address = 47.99.167.137:9076
p2p-peer-address = 47.99.151.178:9076
p2p-peer-address = 116.62.16.248:9076
# producer-name = bp名
# signature-provider = 出块公钥=KEY:出块私钥
```

注意：必须删除所有eosio::wallet_plugin的config.ini配置，及其他无用配置项。如仍有std::exception::what: unrecognised option 报错，可按照错误信息删除相应配置，可仅保留必要配置。

删除钱包插件配置：
```ini
# wallet-dir = "."
# unlock-timeout = 900
```

### 3. 启动

启动新版docker进程

(其他启动文件包含在docker镜像中无需单独下载,  配置目录仅包含config.ini即可。)

```shell
#  需手动建立数据与配置目录 eg: /datatest1.2.0
docker run -d --name test1.2.0 -v /datatest1.2.0/eosforce:/opt/eosio/bin/data-dir -v /datatest1.2.0/nodeos:/root/.local/share/eosio/nodeos -p 9076:9076 -p 19001:19001 eosforce/eos:v1.2.0.test nodeosd.sh

```

查看日志，观察同步或出块是否正常
```shell
docker logs -f --tail 100 test1.2.0
```

----
## 源码编译版本 ：

### 1.下载最新代码并编译

耗时较长建议提几小时操作

```shell
git clone https://github.com/eosforce/eosforce.git eosforce
cd eosforce 
git checkout release
git pull
git submodule update --init --recursive
./eosio_build.sh
```

编译产生的可执行文件(直接使用或拷贝至所需机器目录, 启动节点仅需nodeos):

- eosforce/build/programs/nodeos/nodeos
- eosforce/build/programs/cleos/cleos
- eosforce/build/programs/keosd/keosd

(或 eosforce/build/bin/ 目录下)

### 2.配置启动文件

```shell
configpath='~/eosforcetest/config'

git clone https://github.com/eosforce/dockerfile.git
cd dockerfile
git checkout test.v1.2.0
git pull
cd ..
cp dockerfile/eosforce/genesis.json $configpath

cp build/contracts/System/System.abi build/contracts/System/System.wasm $configpath

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $configpath

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $configpath

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $configpath

```
注意：若使用原有配置目录，注意abi文件有变化，仍需使用最新abi文件替换。

config目录所有必备文件如下：

config.ini  genesis.json

System.abi  System.wasm  

System01.abi  System01.wasm  

eosio.msig.abi  eosio.msig.wasm  

eosio.token.abi  eosio.token.wasm  


### 3.修改配置文件 config.ini

配置p2p地址，端口, bp等，已配则可不变。

```ini
p2p-peer-address = 47.99.138.131:9076
p2p-peer-address = 47.99.165.99:9076
p2p-peer-address = 47.99.167.137:9076
p2p-peer-address = 47.99.151.178:9076
p2p-peer-address = 116.62.16.248:9076
# producer-name = bp名
# signature-provider = 出块公钥=KEY:出块私钥
```

注意：必须删除所有eosio::wallet_plugin的config.ini配置，及其他无用配置项。如仍有std::exception::what: unrecognised option 报错可按照错误信息删除相应配置，可仅保留必要配置。

```ini
# wallet-dir = "."
# unlock-timeout = 900
# required-participation = 33
```

### 4. 启动


```shell
configpath='~/eosforcetest/config' #修改为本地配置文件目录
datapath='~/eosforcetest/data'	#修改为本地数据文件目录
logpath='./nodeostest.log'	#修改为本地日志文件

nohup ./build/programs/nodeos/nodeos --config-dir $configpath --data-dir $datapath > $logpath 2>&1 &
```

查看日志，观察同步或出块是否正常

```shell
tail -100f $logpath
```

------

