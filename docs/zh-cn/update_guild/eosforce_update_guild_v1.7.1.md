# EOSForce v1.7.1 更新说明

更新分为两步: 

1. 通过bp多签更新系统合约
2. 通过bp多签设置removepunish的费用


## . 通过bp多签更新系统合约

查看abi和code多签的内容:

```bash
./cleos --url https://w3.eosforce.cn:443 multisig review xuyapeng p.upsyscode
./cleos --url https://w3.eosforce.cn:443 multisig review xuyapeng p.upsysabi
```

节点批准(这里账户是bpname):

```bash
./cleos --url https://w3.eosforce.cn:443 multisig approve xuyapeng p.upsyscode '{"actor":"bpname","permission":"active"}' -p bpname@active
./cleos --url https://w3.eosforce.cn:443 multisig approve xuyapeng p.upsysabi '{"actor":"bpname","permission":"active"}' -p bpname@active
```

## 5. 通过多签设置removepunish的费用

查看设置removepunish的费用多签的内容
```bash
./cleos --url https://w3.eosforce.cn:443 multisig review xuyapeng p.feermvpsh
```

节点批准(这里账户是bpname):

```bash
./cleos --url https://w3.eosforce.cn:443 multisig approve xuyapeng p.feermvpsh '{"actor":"bpname","permission":"active"}' -p bpname@active
```
