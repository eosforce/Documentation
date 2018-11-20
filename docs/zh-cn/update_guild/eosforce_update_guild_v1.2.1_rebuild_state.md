
## 重建state具体操作步骤：（提前准备好v1.2.1版本 docker镜像或编译好的nodeos）

以默认文件路径 ~/.local/share/eosio/nodeos 为例：
```shell
DATA_PATH=~/.local/share/eosio/nodeos
ls $DATA_PATH
#下面应有：config  data
```

### 1. 暂停原服务
```shell
docker stop '原容器'
#或 
kill -2 原nodeos进程
```

### 2. 拷贝原数据
```shell
DATA_NEW_PATH=~/.local/share/eosio/nodeos-v1.2.0	#可改为自己指定目录
cp -r $DATA_PATH $DATA_NEW_PATH
```

### 3. 重启原服务
```shell
docker start '原容器'
或
nohup 老版本nodeos --config-dir $DATA_PATH'config' --data-dir $DATA_PATH'data' > $log 2>&1 &
```

### 4. 删除state目录
```shell
rm -rf $DATA_NEW_PATH'/data/state/'
```

### 5. 修改config.ini 配置，与abi\wasm文件
```shell
vim $DATA_NEW_PATH'config'
#删除无效配置与bp配置
```
docker版本删除所有abi,wasm文件
```shell
rm $DATA_NEW_PATH'config/*.abi'
rm $DATA_NEW_PATH'config/*.wasm'
```
编译版本替换abi，wasm
```shell
cp build/contracts/System/System.abi build/contracts/System/System.wasm $DATA_NEW_PATH'config'

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $DATA_NEW_PATH'config'

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $DATA_NEW_PATH'config'

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $DATA_NEW_PATH'config'
```

### 6. 启动v1.2.0版本服务, 重建state(耗时1小时左右)

```shell
docker run -d --name eosforce-v1.2.1 -v ~/.local/share/eosio/nodeos-v1.2.0/config:/opt/eosio/bin/data-dir -v ~/.local/share/eosio/nodeos-v1.2.0/data:/root/.local/share/eosio/nodeos -p 8888:8888 -p 9876:9876 eosforce/eos:v1.2.1 nodeosd.sh
```
或
```shell
nohup 新版本nodeos --config-dir $DATA_NEW_PATH'config' --data-dir $DATA_NEW_PATH'data' > $log 2>&1 &
```

### 7. 一切完成后，等到正式升级时，修改config.ini 为正式bp配置，重启
```shell
docker stop ' eosforce-v1.2.1'
vim $DATA_NEW_PATH'config'
docker start ' eosforce-v1.2.1'
#或
kill -2 '新nodeos pid'
nohup 新版本nodeos --config-dir $DATA_NEW_PATH'config' --data-dir $DATA_NEW_PATH'data' > $log 2>&1 &
```
----
## 或者从该链接下载，使用提供的数据文件。
- docker版本： https://updatewallet.oss-cn-hangzhou.aliyuncs.com/eosforce/data/eosforce-data-docker-v1.2.1.tar.gz
- 源码编译版本：https://updatewallet.oss-cn-hangzhou.aliyuncs.com/eosforce/data/bin/eosforce-data-bin-v1.2.1.tar.gz

直接启动v1.2.0版本服务，指定下载的数据文件目录.

其他下载与配置步骤见文档：
https://eosforce.github.io/Documentation/#/zh-cn/update_guild/eosforce_update_guild_v1.2.1
