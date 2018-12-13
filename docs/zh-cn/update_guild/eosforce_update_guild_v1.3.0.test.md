
# 2018-12-06 v1.3.0.test 测试网接入说明

> 使用最新测试版本代码，config.ini配置接入节点，启动即可。 数据从接入节点同步。
> 启动等其他操作与之前一样
>
>  注：需清空data目录数据，启动后需要几小时从接入节点同步历史数据
>
> 我们本次测试使用的数据为截止高度为4705125的线上主网数据进行分叉产生，需要至少2/3(16个)BP接入并开始出块才能完成测试。
> 请节点积极参与测试，如果不能出块请修改此配置 max-irreversible-block-age = -1。

## 接入节点
IP
-    47.99.138.131
-    47.99.165.99


端口
- http: 19000
- p2p: 9076

## 代码

### docker 版本
eosforce/eos:v1.3.0.test

> 配置目录仅放入 config.ini
>

### 编译版本

1. 使用分支编译： release
```shell
git clone https://github.com/eosforce/eosforce.git
cd eosforce
git checkout release
git pull
git submodule update --init --recursive
./eosio_build.sh
```

2. 需要拷贝至config配置目录的文件：

```shell
configpath='~/eosforce/config'

cp Docker/contracts/activeacc.json $configpath

cp build/contracts/eosio.lock/eosio.lock.abi  build/contracts/eosio.lock/eosio.lock.wasm $configpath

cp build/contracts/System/System.abi build/contracts/System/System.wasm $configpath

cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm $configpath

cp build/contracts/eosio.token/eosio.token.abi build/contracts/eosio.token/eosio.token.wasm $configpath

cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm $configpath
```

- genesis.json 文件下载链接：https://updatewallet.oss-cn-hangzhou.aliyuncs.com/eosforce/data/genesis.json

- config.ini 自行创建配置


> config.ini 配置示例

```shell
blocks-dir = "blocks"
chain-state-db-size-mb = 8192
reversible-blocks-db-size-mb = 340
contracts-console = false
filter-on = *
verbose-http-errors = true
http-validate-host = false
https-client-validate-peers = false
http-server-address = 0.0.0.0:19000
access-control-allow-origin = *
access-control-allow-headers = Content-Type
access-control-allow-credentials = false
p2p-listen-endpoint = 0.0.0.0:9076
p2p-peer-address = 172.16.133.35:9076
p2p-max-nodes-per-host = 1
allowed-connection = any
max-clients = 50
connection-cleanup-period = 30
network-version-match = 0
sync-fetch-span = 100
max-implicit-request = 1500
enable-stale-production = false
pause-on-startup = false
max-transaction-time = 1000
max-irreversible-block-age = -1
plugin = eosio::chain_api_plugin
plugin = eosio::history_plugin
plugin = eosio::history_api_plugin
producer-name = sbp.i
signature-provider = EOS8LUba7qUNToMSvHdK6tFvmipY6JDCFubpSKz7C83rMvYwf4ebf=KEY:aaaaaaaaaaaaaaaaaaaaaaaaaaaa
```
## 测试

#### 1. 冻结未激活的创世账号80% EOS，以等量的eosio.lock合约EOSLOCK代币作为凭证。(从0重启后自动运行，节点可验证初始账号EOS余额是否正常)

#### 2. 回归原有功能，并测试资源模型等其他更新(请咨询@范杨)。

#### 3. 修改系统账户active权限为23个超级节点多签账号，以支持链上更新系统合约。
3.1. 指定高度统一更新eosio系统账号的active权限为23个超级节点控制的eosio.prods。(由原力操作)
3.2. 接入并开始出块的BP超过2/3后，提交多钱签名合约提议, 提议内容为："更新系统合约" (由原力或任何其他账户操作)

```shell
cleos  multisig propose pab1 '[{"actor": "eosawake", "permission": "active"}, {"actor": "mathwalletbp", "permission": "active"}, {"actor": "thinkbiteos", "permission": "active"}, {"actor": "hexiaozhang", "permission": "active"}, {"actor": "jiqix", "permission": "active"}, {"actor": "eosou.io", "permission": "active"}, {"actor": "imlianquan", "permission": "active"}, {"actor": "everest", "permission": "active"}, {"actor": "eosgod", "permission": "active"}, {"actor": "eostrust", "permission": "active"}, {"actor": "titanforce", "permission": "active"}, {"actor": "eosshuimu", "permission": "active"}, {"actor": "ccbc", "permission": "active"}, {"actor": "eosio.top", "permission": "active"}, {"actor": "onetaoforce", "permission": "active"}, {"actor": "fos.top", "permission": "active"}, {"actor": "walianwang", "permission": "active"}, {"actor": "eosco", "permission": "active"}, {"actor": "jeepool", "permission": "active"}, {"actor": "eosmainbp", "permission": "active"}, {"actor": "cindydaily", "permission": "active"}, {"actor": "eosecoio", "permission": "active"}, {"actor": "helloforcebp", "permission": "active"}]' '[{"actor":"eosio","permission":"active"}]' eosio setcode '{"account":"eosio","vmtype":0,"vmversion":0,"code":"'$wasm_data'"}' v.test
```

3.3. 23个超级节点使用cleos命令行多签合约命令进行提议的决议，统一提议则执行：

```shell
cleos multisig approve sss pab1 '{"actor":"sbp.a","permission":"active"}' -p sbp.a@active
```
3.4. 超过2/3的超级节点通过决议，则触发执行该多签提议，自动运行更新系统合约。(由原力或任何其他账户操作)

```shell
cleos multisig exec sss pab1 sss
```


