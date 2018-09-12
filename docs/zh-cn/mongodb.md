# eos mongoDB

## 简介
  eosforce v1.1.0版本将支持mongoDB存储。以代替history api插件，解决查询效率低下的问题。

## 快速配置启动
1. 启动mongod, 默认端口：27017。

2. config.ini 配置文件
   
- 添加参数：
mongodb-uri = mongodb://127.0.0.1:27017/EOS

- 添加插件：
plugin = eosio::mongo_db_plugin

1. 启动nodeos. 将通过mongodb-uri连接mongo, 并默认建立库：EOS

## mongoDB collections：

- blocks    区块
- block_states  区块状态
- transactions  交易
- transaction_traces    交易追踪记录
- action_traces 执行动作追踪记录
- accounts  账户
- pub_keys  公钥
- account_controls  控制账户