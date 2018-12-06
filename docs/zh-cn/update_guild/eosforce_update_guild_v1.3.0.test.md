
# 2018-12-06 v1.3.0.test 测试网接入说明

## 接入节点
IP
-    47.99.138.131


端口
- http: 19000
- p2p: 9076

## 代码

### docker 版本
eosforce/eos:v1.3.0.test

> 配置目录仅放入 config.ini
>

### 编译版本

1. 使用分支编译： develop

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

> 启动等其他操作与之前一样
>
>  注：需清空data目录数据，启动后需要几小时从接入节点同步历史数据
