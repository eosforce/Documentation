# eosforce v1.2.1 主网升级说明

#### Tag标签:
v1.2.1

#### 更新说明：
- 合并eosio最新代码
- 优化性能，优化插件与编译器
- 增加延时交易，多重签名合约

docker版本启动 与 源码编译启动方式取一即可，注意两种版本启动节点产生的data文件互不兼容。

注意：需提前备份原数据文件，以防止异常情况污染数据文件。需暂停服务，拷贝后重启；
```shell
docker stop '容器名'
#或 kill -2 'nodeos pid' 

cp -r '原数据目录' '备份数据目录' 

docker start '容器名'
# 或nohup ./build/programs/nodeos/nodeos --config-dir $configpath --data-dir $datapath > $log 2>&1 &
```

#### 常见问题：
- 如果出现 “std::exception::what: unrecognised option” 报错，可按照错误信息删除config.ini中相应的失效配置，可仅保留必要配置。
- 如果出现“memory does not match data expected by executable”此报错， 需要删除数据目录下state文件夹，并启动重建state(预计耗时1小时左右)， 
或直接下载使用新版本数据文件：
- docker版本： https://updatewallet.oss-cn-hangzhou.aliyuncs.com/eosforce/data/eosforce-data-docker-v1.2.1.tar.gz
- 源码编译版本：https://updatewallet.oss-cn-hangzhou.aliyuncs.com/eosforce/data/bin/eosforce-data-bin-v1.2.1.tar.gz

[v1.2.1主网升级重建state操作说明](zh-cn/update_guild/eosforce_update_guild_v1.2.1_rebuild_state.md)

#### get info信息：
```shell
cleos get info
{
  "server_version": "12b3ec59",
  "chain_id": "bd61ae3a031e8ef2f97ee3b0e62776d6d30d4833c8f7c1645c657b149151004b",
	...
  "server_version_string": "v1.2.1"
}
```

-----
## docker 版本：

### 1. 拉取最新版本镜像

```shell
docker pull eosforce/eos:v1.2.1
```

### 2. 修改配置文件 config.ini

其他启动文件已经包含于docker镜像中，无需单独下载。
config配置目录可以仅保留config.ini文件

删除钱包插件等无效配置：
```ini
# wallet-dir = "."
# unlock-timeout = 900
# genesis-json = "/root/.local/share/eosio/nodeos/config/genesis.json"
# get-actions-on = true
# required-participation = 33
```

注意：必须删除所有eosio::wallet_plugin的config.ini配置，及其他无用配置项。
可以预先配置一个空数据目录启动测试，如仍有std::exception::what: unrecognised option 报错，可按照错误信息删除相应配置，可仅保留必要配置。

其他配置可以不变.

### 3. 新建数据与配置目录，并重建state文件

[v1.2.1主网升级重建state操作说明](zh-cn/update_guild/eosforce_update_guild_v1.2.1_rebuild_state.md)

### 4. 启动

停止原节点docker容器

```shell
docker ps
docker stop '原docker容器名'
```

启动新版docker进程

```shell
#下面映射路径 ‘~/eosforce/config’ 需修改为本地配置文件目录；‘~/eosforce/data’ 需修改为本地数据文件目录；
#可修改默认http端口8888 ，p2p网络端口9876
docker run -d --name eosforce -v ~/eosforce/config:/opt/eosio/bin/data-dir -v ~/eosforce/data:/root/.local/share/eosio/nodeos -p 8888:8888 -p 9876:9876 eosforce/eos:v1.2.1 nodeosd.sh
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
cd eosforce
git fetch
git checkout v1.2.1
git submodule update --init --recursive
./eosio_build.sh
```

编译产生的可执行文件(直接使用或拷贝至所需机器目录, 启动节点仅需nodeos):

- eosforce/build/programs/nodeos/nodeos
- eosforce/build/programs/cleos/cleos
- eosforce/build/programs/keosd/keosd

 (或使用 eosforce/build/bin/ 目录下的可执行文件)

### 2.配置启动文件

```shell
configpath='~/eosforce/config'

cp build/contracts/System/System.abi build/contracts/System/System.wasm $configpath

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $configpath

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $configpath

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $configpath

```
注意：若使用原有配置文件目录，注意abi与wasm文件有变化，仍需使用最新文件替换。
genesis.json 文件保持原有文件不变。如需下载，可使用：

```shell
wget https://raw.githubusercontent.com/eosforce/genesis/master/genesis.json -O $configpath'/genesis.json' 
```

config目录所有必备文件如下：

- config.ini
- genesis.json
- System.abi  
- System.wasm  
- System01.abi 
- System01.wasm  
- eosio.msig.abi  
- eosio.msig.wasm  
- eosio.token.abi  
- eosio.token.wasm  


### 3.修改配置文件 config.ini

删除钱包插件等无效配置：
```ini
# wallet-dir = "."
# unlock-timeout = 900
# genesis-json = "/root/.local/share/eosio/nodeos/config/genesis.json"
# get-actions-on = true
# required-participation = 33
```

注意：必须删除所有eosio::wallet_plugin的config.ini配置，及其他无用配置项。
可以预先配置一个空数据目录启动测试，如仍有std::exception::what: unrecognised option 报错，可按照错误信息删除相应配置，可仅保留必要配置。

其他配置可以不变.

#### 4. 重建state操 

[v1.2.1主网升级重建state操作说明](zh-cn/update_guild/eosforce_update_guild_v1.2.1_rebuild_state.md)

### 5. 启动

停止原节点nodeos进程

```shell
ps -aux|grep nodeos
kill -2 '原进程pid'
```

启动nodeos

```shell
configpath='~/eosforce/config' #修改为本地配置文件目录
datapath='~/eosforce/data'	#修改为本地数据文件目录
log='./nodeos.log'	#修改为本地日志文件

nohup ./build/programs/nodeos/nodeos --config-dir $configpath --data-dir $datapath > $log 2>&1 &
```

查看日志，观察同步或出块是否正常

```shell
tail -100f $logpath
```

------

