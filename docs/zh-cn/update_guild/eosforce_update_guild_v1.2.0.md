# eosforce v1.2.0 主网升级说明

合并eosio v1.4.2 代码
优化性能，插件与编译器
增加延时交易，多重签名

（未启动，最新代码未更新）

docker版本启动 与 源码编译启动方式取一即可，注意两种版本启动节点产生的data文件互不兼容。

-----
## docker 版本：

### 1. 拉取最新版本镜像

```shell
docker pull eosforce/eos:v1.2.0
```

其他启动文件已经包含于docker镜像中，无需单独下载。

### 2. 修改配置文件 config.ini

配置p2p地址，bp等，已有可不变。

```ini
p2p-peer-address =IP:端口
producer-name = bp名
signature-provider = 出块公钥=KEY:出块私钥
```

注意：必须删除所有eosio::wallet_plugin的config.ini配置，及其他无用配置项。如仍有std::exception::what: unrecognised option 报错，可按照错误信息删除相应配置，可仅保留必要配置。

删除钱包插件配置：
```ini
# wallet-dir = "."
# unlock-timeout = 900
```

### 3. 启动

停止或删除原节点docker容器

```shell
docker ps
docker stop '原docker容器名'
# docker rm -f '原docker容器名' 
```

备份原数据，防止重启异常污染数据。

```shell
datapath='~/eosforce/data'	#修改为本地数据文件目录
databackuppath='~/eosforce/nodeos_backup'	#修改为本地数据备份文件目录
cp -r $datapath $datapath
```

启动新版docker进程

```shell
#下面映射路径 ‘~/eosforce/config’，‘~/eosforce/data’ 需修改为本地文件目录

docker run -d --name eosforce -v ~/eosforce/config:/opt/eosio/bin/data-dir -v ~/eosforce/data:/root/.local/share/eosio/nodeos -p 8888:8888 -p 9876:9876 eosforce/eos:v1.2.0 nodeosd.sh
```

查看日志，观察同步或出块是否正常
```shell
docker logs -f --tail 100 eosforce
```

----
## 源码编译版本 ：

### 1.下载最新代码并编译

耗时较长建议提前2小时操作

```shell
git clone https://github.com/eosforce/eosforce.git eosforce
git fetch
git checkout v1.2.0
cd eosforce 
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
configpath='~/eosforce/config'
# wget https://raw.githubusercontent.com/eosforce/genesis/master/genesis.json -O $configpath'/genesis.json' #原有文件即可

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

配置p2p地址，bp等，已配则可不变。

```ini
p2p-peer-address =IP:端口
producer-name = bp名
signature-provider = 出块公钥=KEY:出块私钥
```

注意：必须删除所有eosio::wallet_plugin的config.ini配置，及其他无用配置项。如仍有std::exception::what: unrecognised option 报错可按照错误信息删除相应配置，可仅保留必要配置。

```ini
# wallet-dir = "."
# unlock-timeout = 900
# required-participation = 33
```

### 4. 启动

停止原节点nodeos进程

```shell
ps -aux|grep nodeos
kill -2 '原进程pid'
```

备份原数据，防止重启异常污染数据。

```shell
datapath='~/eosforce/data'	#修改为本地数据文件目录
databackuppath='~/eosforce/nodeosbackup'	#修改为本地数据备份文件目录
cp -r $datapath $databackuppath
```

启动nodeos

```shell
configpath='~/eosforce/config' #修改为本地配置文件目录
datapath='~/eosforce/data'	#修改为本地数据文件目录
logpath='./nodeos.log'	#修改为本地日志文件

nohup ./build/programs/nodeos/nodeos --config-dir $configpath --data-dir $datapath > $logpath 2>&1 &
```

查看日志，观察同步或出块是否正常

```shell
tail -100f $logpath
```

------

