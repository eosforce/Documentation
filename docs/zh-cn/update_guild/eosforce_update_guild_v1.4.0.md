# v1.4.0 主网更新操作步骤



## 本次升级增加候选节点签到功能

需要候选节点周期性调用签到action(缺省10分钟签到一次，超过60分钟不签到，则取消该候选节点分红）



## 1. 节点升级 (v1.4.0)

### 配置文件修改

##### 需要变动的配置文件config.ini

- 前23 bp节点保持config.ini 保持不变

- 针对候选节点,有如下改变

config.ini 中需要添加 

plugin = eosio::heartbeat_plugin

bp-mapping=biosbpa=KEY:biosbpaa

(例如bp的名字为biosbpa，需要单独创建一个账号名例如biosbpaa，并公钥和出块的公钥必须相同)

##### 其他配置的文件保持原有不变:

- activeacc.json https://updatewallet.oss-cn-hangzhou.aliyuncs.com/eosforce/activeacc.json (md5sum b7c295454e6e81ab3023baeebf2d9131)
- genesis.json 使用原有文件
	

### docker部署

```
# 容器名：eosforce-v1.4.0
docker pull eosforce/eos:v1.4.0
docker stop 原容器名
docker run -d --name eosforce-v1.4.0 -v 本地配置目录:/opt/eosio/bin/data-dir -v 本地数据目录:/root/.local/share/eosio/nodeos -p 9876:9876 -p 8888:8888 eosforce/eos:v1.4.0 nodeosd.sh
# 查看日志
docker logs -f --tail 100 eosforce-v1.4.0
    
```
验证升级结果, 版本信息：
```shell
docker exec -it eosforce-v1.4.0 opt/eosio/bin/cleos get info
"server_version_string": "force-v1.4.0-1-g0e4e0990b"
```

### 源码编译方式
使用tag: force-v1.4.0

```shell
# 进入eosforce工程目录
git fetch
git checkout force-v1.4.0
git submodule update --init --recursive
./eosio_build.sh
```

编译后使用 build/bin/下生产的可执行文件：cleos  keosd  nodeos

(启动服务仅使用 nodeos)

#### 源码编译方式 需要配置的文件
```shell
configpath='~/eosforce/config' #修改为自己本地服务配置目录

cp build/contracts/eosio.lock/eosio.lock.abi  build/contracts/eosio.lock/eosio.lock.wasm $configpath

cp build/contracts/System/System.abi build/contracts/System/System.wasm $configpath

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $configpath

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $configpath

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $configpath
```

#### 源码编译方式 启动：
启动nodeos

```shell
# 启动
nohup ./build/bin/nodeos --config-dir 配置目录 --data-dir 数据目录 > eos.log 2>&1 &

# 查看日志，观察同步或出块是否正常
tail -100f eos.log
```

#### 编译方式启动后， 验证升级结果
版本信息：
```shell
cleos -u http://127.0.0.1:8888 get info
"server_version_string": "force-v1.4.0-1-g0e4e0990b"
```




## 2. 多签更新系统合约


