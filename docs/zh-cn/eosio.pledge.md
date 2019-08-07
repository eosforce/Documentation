# eosio.pledge说明

eosio.pledge 是一个记录抵押信息的合约，可以让其他合约调用并查询相关帐户的抵押情况。

## eosio.pledge 提供的功能

- addtype       添加抵押类别
- addpledge     增加抵押
- deduction     扣除押金
- withdraw      领取抵押
- getreward     领取奖励
- allotreward   分发奖励
- open          创建抵押条目
- close         删除空的抵押条目

### addtype

相关参数
- pledge_name           抵押类别名称
- deduction_account     拥有处理抵押权力的帐户
- ram_payer             内存支付者
- quantity              抵押收取的币种
- memo                  备注

示例
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge addtype '["block.out","eosio","eosio","2.0000 EOS","system pledge"]' -p eosio
```

注意：目前addtype只允许eosio帐户使用，其他帐户并没有使用权限

### addpledge

相关参数
- from                  添加抵押的帐户
- to                    合约帐户
- quantity              抵押的金额
- memo                  抵押类别名称

示例
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge open '["block.out","eosforce","testopen"]' -p eosforce
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 transfer eosforce eosio.pledge "20000.0000 EOS" "block.out"
```

注意：addpledge是监控eosio的transfer，如果transfer的to是eosio.pledge就会调用addpledge，用户无法自行调用，如果是第一次调用需要先调用open将抵押条目增加出来，抵押金额才能更新到抵押条目上面

### deduction

相关参数
- pledge_name           抵押类别名称
- debitee               扣除抵押的帐户名
- quantity              扣除抵押的金额
- memo                  备注

示例
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge deduction '["block.out","biosbpa","1.0000 EOS","test deduction"]' -p eosio
```

注意：只有deduction_account帐户才有扣除抵押的权限

### withdraw

- pledge_name           抵押类别名称
- pledger               领取抵押的帐户
- quantity              领取的金额
- memo                  备注

示例
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge withdraw '["block.out","eosforce","1.0000 EOS","test withdraw"]' -p eosforce
```

注意：用户只能领取自己抵押的金额，被deduction_account扣除的部分不能领取

### getreward

相关参数
- rewarder              领取奖励的帐户
- quantity              领取奖励的币种
- memo                  备注

示例
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge getreward '["eosforce","5.0000 EOS","test reward"]' -p eosforce
```

注意：每次领取都会领取所有的奖励，quantity只取币种

### allotreward

相关参数
- pledge_name           抵押类别名称
- pledger               抵押币的帐户
- rewarder              奖励币的帐户
- quantity              奖励的金额
- memo                  备注

示例
```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge allotreward '["block.out","eosforce","eosforce","1.0000 EOS","test allotreward"]' -p eosio
```

注意：只有deduction_account帐户才有分发奖励的功能，分发奖励的金额不得超过pledger被扣除的金额。

### open

相关参数
- pledge_name           抵押类别名称
- payer                 开辟抵押条目的帐户名
- memo                  备注

```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge open '["block.out","eosforce","testopen"]' -p eosforce
```

注意：每个帐户在一个抵押类别上只需要open一次，多次open不会报错。

### close

相关参数
- pledge_name           抵押类别名称
- payer                 开辟抵押条目的帐户名
- memo                  备注

```bash
cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 push action eosio.pledge close '["block.out","eosforce","testopen"]' -p eosforce
```

注意：只有没有抵押和没有扣款的条目才能被close掉

## eosio 多签调用addtype

发起多签
```bash
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig propose pledge1 '[{"actor":"biosbpa","permission":"active"},{"actor":"biosbpb","permission":"active"},{"actor":"biosbpc","permission":"active"},{"actor":"biosbpd","permission":"active"},{"actor":"biosbpe","permission":"active"},{"actor":"biosbpf","permission":"active"},{"actor":"biosbpg","permission":"active"},{"actor":"biosbph","permission":"active"},{"actor":"biosbpi","permission":"active"},{"actor":"biosbpj","permission":"active"},{"actor":"biosbpk","permission":"active"},{"actor":"biosbpl","permission":"active"},{"actor":"biosbpm","permission":"active"},{"actor":"biosbpo","permission":"active"},{"actor":"biosbpp","permission":"active"},{"actor":"biosbpq","permission":"active"},{"actor":"biosbpr","permission":"active"},{"actor":"biosbps","permission":"active"},{"actor":"biosbpt","permission":"active"},{"actor":"biosbpu","permission":"active"},{"actor":"biosbpn","permission":"active"}]' '[{"actor":"eosio","permission":"active"}]' eosio.pledge addtype '{"pledge_name":"block.out","deduction_account":"eosio","ram_payer":"eosio","quantity":"2.0000 EOS","memo":"system pledge"}' eosforce
```

同意多签
```bash
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpb","permission":"active"}' -p biosbpb@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpa","permission":"active"}' -p biosbpa@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpc","permission":"active"}' -p biosbpc@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpd","permission":"active"}' -p biosbpd@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpe","permission":"active"}' -p biosbpe@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpf","permission":"active"}' -p biosbpf@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpg","permission":"active"}' -p biosbpg@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbph","permission":"active"}' -p biosbph@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpi","permission":"active"}' -p biosbpi@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpj","permission":"active"}' -p biosbpj@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpk","permission":"active"}' -p biosbpk@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpl","permission":"active"}' -p biosbpl@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpm","permission":"active"}' -p biosbpm@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpn","permission":"active"}' -p biosbpn@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpo","permission":"active"}' -p biosbpo@active
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig approve eosforce pledge1 '{"actor":"biosbpp","permission":"active"}' -p biosbpp@active
```

执行多签
```bash
../../build/programs/cleos/cleos --wallet-url http://127.0.0.1:6666 --url http://127.0.0.1:8001 multisig exec eosforce pledge1 eosforce
```

注意：biosbpa~biosbpu是出块节点，真实链上操作的时候需要将其修改为真是的出块节点






