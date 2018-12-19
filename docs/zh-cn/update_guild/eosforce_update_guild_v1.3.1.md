# v1.3.1 主网更新操作步骤

注意：每一步要等其他节点操作完成后，再进行下一步操作。

## 1. 设置链紧急状态, 暂停链上action执行。（需要超过2/3节点设置后生效）
```shell
cleos -u https://w1.eosforce.cn push action eosio setemergency '["BP账户名", true]' -p BP账户名@active
```
#### 查看自己是否成功设置，字段为 "emergency": 1
```shell
cleos -u https://w1.eosforce.cn get table eosio eosio bps -k bp账号名
```
#### 查看链紧急状态是否设置成功, 需要超过2/3节点设置后生效
```shell
cleos -u https://w1.eosforce.cn get table eosio eosio chainstatus
```

暂停主网后，我们会统计截止此时的所有激活的创世账号(约2小时)，生成activeacc.json，BP节点以此名单为准启动，冻结所有未激活的创世账号80%EOS.
并创建包含此名单的docker v1.3.1镜像。供下一步升级使用。
若使用源码编译方式启动的，需要使用我们之后提供的链接下载。
(所有人可以核查此名单)

## 2. 节点升级，新启动一个同步节点 (v1.3.1)	

**必须新建空的本地数据目录启动，重新通过主网节点同步数据** (从空数据启动才会重新初始化创世账号，冻结未激活创世账号)(**可能需要5小时同步,或等待原力同步完成后发布下载数据包**)

### docker方式
```shell
docker pull eosforce/eos:v1.3.1
docker stop 容器名
docker rm -f 容器名
docker run -d --name 容器名 -v 本地配置目录:/opt/eosio/bin/data-dir -v 空的本地数据目录:/root/.local/share/eosio/nodeos -p 9876:9876 -p 8888:8888 eosforce/eos:v1.3.1 nodeosd.sh
docker start 容器名
# 查看日志
docker logs -f --tail 100 容器名
```
需检查配置目录中activeacc.json是否正确。(完成第1步后会提供md5sum值)

### 源码编译方式：使用tag: force-v1.3.1 

```shell
# 进入eosforce工程目录
git fetch
git checkout force-v1.3.1
git submodule update --init --recursive
./eosio_build.sh
```
编译后使用 build/bin/下生产的可执行文件：cleos  keosd  nodeos

(启动服务仅使用 nodeos)

#### 需要配置的文件
```shell
configpath='~/eosforce/config' #修改为自己本地服务配置目录

cp build/contracts/eosio.lock/eosio.lock.abi  build/contracts/eosio.lock/eosio.lock.wasm $configpath

cp build/contracts/System/System.abi build/contracts/System/System.wasm $configpath

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $configpath

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $configpath

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $configpath
```
其他需要配置的文件:
- activeacc.json 使用我们第1步完成之后提供的链接下载。需检查配置目录中activeacc.json是否正确。(完成第1步后会提供md5sum值)
- genesis.json 使用原有文件
- config.ini 使用原有文件或自行创建配置

#### 启动：
启动nodeos

```shell
configpath='~/eosforce/config' #修改为本地配置文件目录
datapath='~/eosforce/data'	#修改为本地数据文件目录(必须是空的)
log='./nodeos.log'	#修改为本地日志文件

nohup ./build/bin/nodeos --config-dir $configpath --data-dir $datapath > $log 2>&1 &

# 查看日志，观察同步或出块是否正常
tail -100f $logpath
```

### 验证升级结果
版本信息：
```shell
cleos -u http://127.0.0.1:8888 get info
"server_version_string": "force-v1.3.1"
```
### 同步完成后备份现有数据
```shell
# 暂停原服务
docker stop xxx
# kill -2 xxx
cp -r 数据目录 备份目录
# 重启服务
docker start
# nohup ./build/bin/nodeos --config-dir $configpath --data-dir $datapath > $log 2>&1 &
```

### 同步节点完成后，将原BP配置(包括出块公私钥对)移至新节点的配置文件中，重启。关闭老bp节点。

> 所有节点完成升级后，必须等待统一指令再执行后续步骤。

## 3. 升级完成后，恢复‘链紧急状态’，统一指令再执行
```shell
cleos -u https://w1.eosforce.cn push action eosio setemergency '["BP账户名", false]' -p BP账户名@active
```
检查正常操作是否恢复执行

## 4. 多签更新系统合约

原力force.msig账号发起多签提议后，节点执行(需要使用命令行创建钱包导入节点账户私钥)：

```shell
# 批准更新系统合约code多签提议
cleos  -u https://w1.eosforce.cn multisig approve force.msig p.upsyscode '{"actor":"节点账户名","permission":"active"}' -p 节点账户名@active
# 批准更新系统合约abi多签提议
cleos  -u https://w1.eosforce.cn multisig approve force.msig p.upsysabi '{"actor":"节点账户名","permission":"active"}' -p 节点账户名@active
```
超过2/3节点执行通过，即可执行多签更新系统合约。



## 验证冻结未激活的创世账号执行强开, (冻结80% EOS，以等量的eosio.lock合约EOSLOCK代币作为凭证。)
调用自己机器的服务，查看是否冻结，是否升级成功。
```shell
cleos -u http://127.0.0.1:8888 get table eosio eosio accounts -k ge3tegenesis
cleos -u http://127.0.0.1:8888 get table eosio.lock eosio.lock accounts -k ge3tegenesis
```

