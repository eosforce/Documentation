# v1.6.0 主网更新操作步骤


### 升级内容：

1. 合并EOSIO 1.7.4版本
2. 完善链上配置功能, 便于后续更新
3. 部分代码优化





## 1. 节点升级 (v1.6.0)

#### 所有节点升级顺序，先完成bp节点升级，后升级同步节点

#####提前准备工作
本次升级配置有所改变，config.ini文件 需要删除相关一些选项：

1. max-implicit-request
2. txn-reference-block-lag
3. reversible-blocks-db-size-mb
4. access-control-allow-credentials
5. connection-cleanup-period
6. network-version-match
7. sync-fetch-span
8. enable-stale-production
9. pause-on-startup
10. keosd-provider-timeout

 

### docker部署

```
# 容器名：eosforce-v1.6.0
docker pull eosforce/eos:v1.6.0
docker stop 原容器名
docker run -d --name eosforce-v1.6.0 -v 本地配置目录:/opt/eosio/bin/data-dir -v 本地数据目录:/root/.local/share/eosio/nodeos -p 9876:9876 -p 8888:8888 eosforce/eos:v1.6.0 nodeosd.sh
# 查看日志
docker logs -f --tail 100 eosforce-v1.6.0
    
```
验证升级结果, 版本信息：
```shell
docker exec -it eosforce-v1.6.0 opt/eosio/bin/cleos get info
"server_version_string": "force-v1.6.0"
```

### 源码编译方式
使用tag: force-v1.6.0 


```shell
# 进入eosforce工程目录

git fetch
git checkout force-v1.6.0
git submodule update --init --recursive
./eosio_build.sh
```

注意事项（如果编译出错，需要删除HOME目录下的opt目录，重新编译）

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
"server_version_string": "force-v1.6.0"
```




## 2. 多签更新系统合约

原力 eosio.msig 账号发起多签提议后，节点执行(需要使用命令行创建钱包导入节点账户私钥)：

```shell
# 批准更新系统合约code多签提议
cleos -u http://47.99.138.131:8888 multisig approve force.msig f.votagen '{"actor":"节点账户名","permission":"active"}' -p 节点账户名@active

# 批准更新系统合约abi多签提议
cleos -u http://47.99.138.131:8888  multisig approve force.msig  f.cprod '{"actor":"节点账户名","permission":"active"}' -p 节点账户名@active
```
超过2/3节点执行通过，即可执行多签更新系统合约。

```shell
# 查看提议批准情况
cleos  -u http://47.99.138.131:8888 get table eosio.msig force.msig approvals
```
