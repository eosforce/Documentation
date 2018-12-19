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

暂停主网后，我们会统计截止此时的所有激活的创世账号(约1小时)，生成activeacc.json，BP节点以此名单为准启动，冻结所有未激活的创世账号80%EOS.
并创建包含此名单的docker v1.3.1镜像。供下一步升级使用。
若使用源码编译方式启动的，需要使用我们之后提供的链接下载。
(所有人可以核查此名单)

## 2. 节点升级 (v1.3.1)	

> 等待第一步完成后发布activeacc.json文件，或包含最新activeacc.json文件的docker镜像 后进行。

### 数据文件说明: **必须删除state文件夹重建state, 或新建空的本地数据目录启动重新同步**  (从空state数据启动重新初始化创世账号，冻结未激活创世账号部分余额)
>  由于本地升级block数据可兼容，state数据需重建，拷贝原数据后删除其中state文件夹数据，用新版程序启动重建state。(约1~2小时)
> 其他方法
> 1. 新启动同步节点，config.ini仅配置自己的老节点的p2p端口，从本机老节点同步数据。同步好后，将原节点的bp配置移至新节点上，重启即可。(约3-4小时)
> 2. 或等待原力同步完成后发布数据包(约3-4小时), 下载并使用此数据包启动。

### 新启动一个同步节点: docker方式
```shell
# 容器名：eosforce-v1.3.1
docker stop 原有容器
# 拷贝数据
cp -r 原有数据文件夹 新数据文件夹
# 删除state，以重建
rm -rf 新数据文件夹/state/
docker pull eosforce/eos:v1.3.1
docker run -d --name eosforce-v1.3.1 -v 本地配置目录:/opt/eosio/bin/data-dir -v 新数据目录:/root/.local/share/eosio/nodeos -p 9876:9876 -p 8888:8888 eosforce/eos:v1.3.1 nodeosd.sh
docker start eosforce-v1.3.1
# 查看日志
docker logs -f --tail 100 eosforce-v1.3.1
```
验证升级结果, 版本信息：
```shell
docker exec -it eosforce-v1.3.1 opt/eosio/bin/cleos get info
"server_version_string": "force-v1.3.1"
```
需检查配置目录中activeacc.json是否正确。(完成第1步后会提供md5sum值)

### 新启动一个同步节点: 源码编译方式
使用tag: force-v1.3.1 

```shell
# 进入eosforce工程目录
git fetch
git checkout force-v1.3.1
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
其他需要配置的文件:
- activeacc.json 使用我们第1步完成之后提供的链接下载。需检查配置目录中activeacc.json是否正确。(完成第1步后会提供md5sum值)
- genesis.json 使用原有文件
- config.ini 使用原有文件或自行创建配置

#### 源码编译方式 启动：
启动nodeos

```shell
# 停止原nodeos进程
kill -2 原nodeos进程pid
# 拷贝数据
cp -r 原有数据文件夹 新数据文件夹
# 删除state，以重建
rm -rf 新数据文件夹/state/
# 启动
nohup ./build/bin/nodeos --config-dir 配置目录 --data-dir 新数据目录 > eos.log 2>&1 &

# 查看日志，观察同步或出块是否正常
tail -100f eos.log
```

#### 编译方式启动后， 验证升级结果
版本信息：
```shell
cleos -u http://127.0.0.1:8888 get info
"server_version_string": "force-v1.3.1"
```
### 同步完成后备份现有数据
```shell
# 暂停原服务
docker stop xxx
# 或 kill -2 xxx
cp -r 数据目录 备份目录
# 重启服务
docker start
# 或 nohup ./build/bin/nodeos --config-dir 配置目录 --data-dir 数据目录 > eos.log 2>&1 &
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

```shell
# 查看提议批准情况
cleos  -u https://w1.eosforce.cn get table eosio.msig force.msig approvals
```


## 验证冻结未激活的创世账号执行强开, (冻结80% EOS，以等量的eosio.lock合约EOSLOCK代币作为凭证。)
调用自己机器的服务，查看是否冻结，是否升级成功。
```shell
cleos -u http://127.0.0.1:8888 get table eosio eosio accounts -k ge3tegenesis
cleos -u http://127.0.0.1:8888 get table eosio.lock eosio.lock accounts -k ge3tegenesis
```

