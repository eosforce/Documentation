# EOSForce v1.7.1 Release Notes

Update is divided into two steps: 

1. Update system contract with bp multi-sign
2. Set the cost of removepunish by bp multi-sign


## . Update system contract with bp multi-sign

Here code and abi need to be approved separately:

```bash
./cleos --url https://w3.eosforce.cn:443 multisig review xuyapeng p.upsyscode
./cleos --url https://w3.eosforce.cn:443 multisig review xuyapeng p.upsysabi
```

Node approval (here the account is bpname):

```bash
./cleos --url https://w3.eosforce.cn:443 multisig approve xuyapeng p.upsyscode '{"actor":"bpname","permission":"active"}' -p bpname@active
./cleos --url https://w3.eosforce.cn:443 multisig approve xuyapeng p.upsysabi '{"actor":"bpname","permission":"active"}' -p bpname@active
```

## 5. Set the cost of removepunish by multi-signing

```bash
./cleos --url https://w3.eosforce.cn:443 multisig review xuyapeng p.feermvpsh
```

Node approval (here the account is bpname):

```bash
./cleos --url https://w3.eosforce.cn:443 multisig approve xuyapeng p.feermvpsh '{"actor":"bpname","permission":"active"}' -p bpname@active
```
