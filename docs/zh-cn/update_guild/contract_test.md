# 合约测试说明

这次我们测试网会开放合约部署，以下是具体的操作说明：

## 1. 部署合约

在EOSForce中部署合约和EOS相同，这里以Hello合约为例：

先创建合约账户：

```
cleos create account eosforce hello EOS8hx7eQtUEFbejmWdatuwS8ZDMQmkhZyhiQkqhztK8HJMHgAQe8 EOS8hx7eQtUEFbejmWdatuwS8ZDMQmkhZyhiQkqhztK8HJMHgAQe8 -p eosforce
```

部署合约需要手续费，为这个账户转一些EOS：

```
cleos push action eosio transfer '{"from":"eosforce","to":"hello","quantity":"100000.0000 EOS","memo":""}' -p eosforce
```

测试网中EOSC可以联系原力队长获得。

注意部署合约需要hello账户权限，这里把私钥导入：

```
cleos wallet import --private-key 5K7DmCWxMnqtAPraoPVGWFvQ1r3BrbRnHoPPRv3UxNGNixUAudv
```

部署合约：

```
cleos set contract hello ../../contracts/hello
```

这里部署的是hello合约，成功会有以下提示：

```
Setting ABI...
executed transaction: f283317d8808cd7591cb14128b97cc74fa279aafcba42f4de88b55e28ca48065  2472 bytes  2735 us
#         eosio <= eosio::setabi                "00000000001aa36a912b0e656f73696f3a3a6162692f312e30000102686900010475736572046e616d65010000000000008...
warning: transaction executed locally, but may not be confirmed by the network yet    ] 
Reading WASM from ../../contracts/hello/hello.wasm...
Setting Code...
executed transaction: 1837d9fe2774ae32c5be7a41de8bdf37d800c30112698145aa8ead227dde5858  1752 bytes  3450 us
#         eosio <= eosio::setcode               "00000000001aa36a0000e2170061736d01000000013b0c60027f7e006000017e60027e7e0060027f7f006000017f60027f7...
warning: transaction executed locally, but may not be confirmed by the network yet    ] 
```

需要注意的是测试网暂时不允许更新合约。

## 2. 设置手续费

在EOSForce临时的合约实现中需要先设置手续费才能执行合约，上述部署的合约如果直接调用会出现错误：

```
cleos push action hello hi '{"user":"abc"}' -p eosforce 
Error 3050000: Action validate exception
```

详细的报错信息是这样的：

```
Exception Details: 3050000 action_validate_exception: Action validate exception
action hello hi name not include in feemap or db
    {"acc":"hello","act":"hi"}
    thread-0  txfee_manager.cpp:84 get_required_fee
```

这是因为系统获取不到手续费信息，需要调用setfee action来设置fee，在测试网中setfee的权限是公开的，先导入对应的私钥：

```
cleos wallet import --private-key 5K7DmCWxMnqtAPraoPVGWFvQ1r3BrbRnHoPPRv3UxNGNixUAudv
```

可以使用cleos 设置action的手续费，注意一个合约下的action需要单独设置，如hello的hi action，我们设置手续费：

```
cleos set setfee hello hi "1.0000 EOS" 6400 1000 64
```

setfee 命令使用方式如下：

```
Set Fee need to action
Usage: ./cleos set setfee [OPTIONS] account action fee cpu_limit net_limit ram_limit

Positionals:
  account TEXT                The account to set the Fee for (required)
  action TEXT                 The action to set the Fee for (required)
  fee TEXT                    The Fee to set to action (required)
  cpu_limit UINT              The cpu max use limit to set to action (required)
  net_limit UINT              The net max use limit to set to action (required)
  ram_limit UINT              The ram max use limit to set to action (required)
```

其中

- account 合约账户名
- action 要设置手续费的action名
- fee 手续费，只接受eos
- cpu_limit 每次执行合约使用的cpu资源上限，单位为us
- net_limit 每次执行合约使用的net资源上限，单位为byte
- ram_limit 每次执行合约使用的ram资源上限，单位为byte

## 3. 执行合约

设置手续费之后，就可以使用合约：

```
 cleos push action hello hi '{"user":"abc"}' -p eosforce
```

如果执行时资源触及上限，则会报以下错误：

```
Error 3040000: Transaction exception
Ensure that your transaction satisfy the contract's constraint!
```

## 4. 相关注意事项

需要注意的是，目前EOSForce不支持延迟合约，也不支持上下文无关合约和在合约中调用上下文无关合约。
合约相关可以参见 https://developers.eos.io/eosio-cpp/v1.2.0/docs/introduction
