# eosforce v1.2.0 主网升级说明

（未启动，最新代码未更新）

docker版本启动 与 源码编译启动方式取一即可，注意两种版本启动节点产生的data文件互不兼容。

-----
## docker 版本：

### 1. 拉取最新版本镜像

```shell
docker pull eosforce/eos:v1.2.0
```

### 2. 下载启动文件

建立配置目录，下载启动所需文件：

```shell
git clone https://github.com/eosforce/dockerfile
```
注意：若使用原有配置目录，注意abi文件有变化，仍需下载最新abi文件替换。

eg:
```shell
config_path='~/.local/share/eosio/nodeos/config'	#修改为本地配置文件目录
cp eosforce/config.ini $config_path
cp eosforce/genesis.json $config_path
cp eosforce/*.abi $config_path
cp eosforce/*.abi $config_path
```
config目录所有必备文件如下：

config.ini  genesis.json
System.abi  System.wasm  
System01.abi  System01.wasm  
eosio.bios.abi  eosio.bios.wasm  
eosio.msig.abi  eosio.msig.wasm  
eosio.token.abi  eosio.token.wasm  

### 3. 修改配置文件 config.ini

配置p2p地址，bp等，已有可不变。

```ini
p2p-peer-address =IP:端口
producer-name = bp名
signature-provider = 出块公钥=KEY:出块私钥
```

注意：必须删除所有eosio::wallet_plugin的config.ini配置，及其他无用配置项。如仍启动报错可按照错误信息删除相应配置，可仅保留必要配置。

删除钱包插件配置：
```ini
# wallet-dir = "."
# unlock-timeout = 900
```

### 4. 启动

停止或删除原节点docker进程
```shell
docker ps
docker stop '原进程名'
# docker rm -f '原进程名' 
```

启动新版docker进程
```shell
config_path='~/.local/share/eosio/nodeos/config' #修改为本地配置文件目录
data_path='~/.local/share/eosio/nodeos/data'	#修改为本地数据文件目录

docker run -d --name eosforce-v1.2.0 -v $config_path':/opt/eosio/bin/data-dir' -v $data_path':/root/.local/share/eosio/nodeos' -p 8888:8888 -p 9876:9876 eosforce/eos:v1.2.0 nodeosd.sh
```

查看日志，观察同步或出块是否正常
```shell
docker logs -f --tail 100 eosforce-v1.2.0
```

----
## 源码编译版本 ：

### 1.下载最新代码并编译

耗时较长建议提前半天操作

```shell
git clone https://github.com/eosforce/eosforce.git eosforce
cd eosforce 
git submodule update --init --recursive
./eosio_build.sh
```

### 2.创建配置目录并复制启动文件

```shell
config_path='~/.local/share/eosio/nodeos/config'

cp build/contracts/System/System.abi build/contracts/System/System.wasm $config_path

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $config_path

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $config_path

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $config_path

cp build/contracts/eosio.bios/eosio.bios.abi build/contracts/eosio.bios/eosio.bios.wasm $config_path

```
注意：若使用原有配置目录，注意abi文件有变化，仍需使用最新abi文件替换。

config目录所有必备文件如下：

config.ini  genesis.json

System.abi  System.wasm  

System01.abi  System01.wasm  

eosio.bios.abi  eosio.bios.wasm  

eosio.msig.abi  eosio.msig.wasm  

eosio.token.abi  eosio.token.wasm  


### 3.修改配置文件 config.ini

配置p2p地址，bp等，已配则可不变。

```ini
p2p-peer-address =IP:端口
producer-name = bp名
signature-provider = 出块公钥=KEY:出块私钥
```

注意：必须删除所有eosio::wallet_plugin的config.ini配置，及其他无用配置项。如仍启动报错可按照错误信息删除相应配置，可仅保留必要配置。

删除钱包插件配置：

```ini
# wallet-dir = "."
# unlock-timeout = 900
```

### 4. 启动

停止原节点nodeos进程

```shell
ps -aux|grep nodeos
kill -2 '原进程pid'
```

启动新版docker进程

```shell
config_path='~/.local/share/eosio/nodeos/config' #修改为本地配置文件目录
data_path='~/.local/share/eosio/nodeos/data'	#修改为本地数据文件目录
log_path='./nodeos.log'	#修改为本地日志文件

nohup ./build/programs/nodeos/nodeos --config-dir $config_path --data-dir $data_path > $log_path 2>&1 &
```

查看日志，观察同步或出块是否正常

```shell
tail -100f $log_path
```

------
