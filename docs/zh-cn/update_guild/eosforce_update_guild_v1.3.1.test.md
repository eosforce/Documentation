# v1.3.1.test 测试网演练操作步骤

## 1. ‘设置链紧急状态’, 暂停链上action执行
```shell
cleos -u http://47.99.138.131:19000 push action eosio setemergency '["BP账户名", true]' -p BP账户名@active
```
#### 查看自己是否成功设置，字段为 "emergency": 1
```shell
cleos -u http://47.99.138.131:19000 get table eosio eosio bps -k bp账号名
```
#### 查看链紧急状态是否设置成功, 需要超过2/3节点设置后生效
```shell
cleos -u http://47.99.138.131:19000 get table eosio eosio chainstatus
```

## 2. 节点升级，在原测试网机器上操作，更新原测试节点 (v1.3.1.test有更新)
```shell
docker pull eosforce/eos:v1.3.1.test
docker stop 容器名
docker rm -f 容器名
docker run -d --name 容器名 -v 本地配置目录:/opt/eosio/bin/data-dir -v 本地数据目录:/root/.local/share/eosio/nodeos -p 9076:9076 -p 19000:19000 eosforce/eos:v1.3.1.test nodeosd.sh
docker start 容器名
```

## 3. 验证：冻结未激活的创世账号80% EOS，以等量的eosio.lock合约EOSLOCK代币作为凭证。
```shell
cleos -u http://47.99.138.131:19000 get table eosio eosio accounts -k ge3tegenesis
cleos -u http://47.99.138.131:19000 get table eosio.lock eosio.lock accounts -k ge3tegenesis
```
## 4. 多签更新系统合约
原力发起多签提议后，节点执行(需要使用命令行创建钱包导入节点账户私钥)：
```shell
cleos multisig approve v.test p.upsyscode '{"actor":"节点账户名","permission":"active"}' -p 节点账户名@active
```
超过2/3节点执行通过，即可执行多签更新系统合约。

## 5. 多签设置块高度生效
原力发起多签提议后，节点执行(需要使用命令行创建钱包导入节点账户私钥)：
```shell
cleos multisig approve v.test p.blocknum '{"actor":"节点账户名","permission":"active"}' -p 节点账户名@active
```
超过2/3节点执行通过，即可执行。

## 6. 升级完成后，恢复‘链紧急状态’
```shell
cleos -u http://47.99.138.131:19000 push action eosio setemergency '["BP账户名", false]' -p BP账户名@active
```




