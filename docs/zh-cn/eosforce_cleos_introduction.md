# Cleos说明

cleos是一个eos的命令行工具，封装了对nodeos的REST接口的调用，更方便使用。

cleos可通过编译eosforce项目后，在./build/program/cleos目录下得到。

使用cleos需要连接到一个nodeos的节点(IP地址:端口)，默认是本地8888端口。


## 测试链

- IP: 47.98.249.86

- Port：8888

使用示例: 

```bash
cleos -u http://47.98.249.86:8888 get info
```
> 测试多签需使用此代码编译的cleos： https://github.com/eosforce/eosforce/tree/feature/updateauth-msig

## cleos命令参数

cleos [选项] 子命令
```
选项:
  -h,--help                   打印帮助信息
  -u,--url TEXT=http://localhost:8888/
                              运行的nodeos的http/https URL
  --wallet-url TEXT=http://localhost:8900/
                              钱包管理器的http/https URL
  -r,--header                 http头
  -n,--no-verify              当使用https时不验证节点权限
  -v,--verbose                错误信息可视输出

Subcommands:
  version                     版本信心
  create                      创建变量、链开关
  get                         获取信息
  set                         设置修改链状态
  transfer                    (无效子命令)转账，是系统代币，eosforce无效
  net                         管理本地p2p网络
  wallet                      管理本地钱包
  sign                        交易签名
  push                        提交一个二进制交易到链上
  [multisig](zh-cn/contract/eosio.msig/msig.md)                   多重签名，需使用eosforce最新代码编译cleos才能使用
  system                      (无效子命令)发送 eosio.system系统合约action到链上，我们没有使用此合约
```

## 常用功能cleos命令
### 1. 获取账号余额:

```shell
./cleos get table eosio eosio accounts -k 账户名
```

> 从系统合约accounts表中读取

```json
{
  "rows": [{
      "name": "b1",
      "available": "103241.0000 EOS"
    }
  ],
  "more": false
}
```

### 2. 通过公钥获取其所有账号

```shell
./cleos get accounts 公钥
```

### 3. 转账

```shell
./cleos push action eosio transfer '{"from":"bbb","to":"aaa", "quantity":"1000.0000 EOS", "memo":""}' -p bbb@active
```

> 提交System系统合约转账交易

### 4. 投票

> 提交System系统合约投票交易

### 5. 分红

> 提交System系统合约分红交易