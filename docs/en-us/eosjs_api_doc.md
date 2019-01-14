# EOSJS API 文档

-----------------------------

## Install

if you are using npm 
```bash
npm install eosforce
npm install bignumber.js //处理javascript大数字处理精准度不够的问题
```
if you are using yarn
```bash
yarn add eosforce
yarn add bignumber.js //处理javascript大数字处理精准度不够的问题
```

## Base Config

```javascript
{
    httpEndpoint: node_rpc_url, //节点rpc端口基础路径,如 http://192.168.82.173:8888
    keyProvider: privateKey, //私钥,部分接口需要
    chainId: 'chainId'//部分接口需要, 通过getBlock获得
}
```

## Example


```javascript
import Eos from 'eosforce';
const config = { 
    httpEndpoint: node_rpc_url //节点rpc端口基础路径,如 http://192.168.82.173:8888
}
//获取节点基本信息
const test_get_info = async () => {
    let res = await  Eos(config).getInfo({});
    console.log(res);
}

test_get_info()
```

## Api

### getInfo

get node info

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |

returnn：

| 字段                          | 说明         |
| --------------------------- | ---------- |
| block_cpu_limit             |            |
| block_net_limit             |            |
| chain_id                    |            |
| head_block_id               | 当前区块最高高度id |
| head_block_num              | 当前区块最高高度数量 |
| head_block_producer         |            |
| head_block_time             |            |
| last_irreversible_block_id  |            |
| last_irreversible_block_num |            |
| server_version              |            |
| virtual_block_cpu_limit     |            |
| virtual_block_net_limit     |            |


```javascript
 const test_get_info = async () => {
    let res = await  Eos(config).getInfo({});
    console.log(res);
 }
 
 test_get_info()

```
### getBlock

get block infomation by block num or block id
params:

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |

return:

| 字段                 | 说明                                           |
| ------------------ | -------------------------------------------- |
| action_mroot       |                                              |
| block_extensions   |                                              |
| block_num          | 区块总数中的序列                                     |
| confirmed          |                                              |
| header_extensions  |                                              |
| id                 |                                              |
| new_producers      |                                              |
| previous           |                                              |
| producer           | 创建者                                          |
| producer_signature |                                              |
| ref_block_prefix   |                                              |
| schedule_version   | 投票届数，若是是当前最高区块，可以获得当前是第几届投票届.最高区块通过getInfo获得 |
| timestamp          |                                              |
| transaction_mroot  |                                              |

```javascript
const test_get_block = async () => {
    let result = await Eos({ httpEndpoint: node_rpc_url}).getBlock({ block_num_or_id: 100 });
    console.log(result);
}

test_get_block()
```
    
### getKeyAccounts

get accounts though key

| parmas       | des                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| public_key   | 根据私钥算出来的公钥                               |



```javascript
import EOS from 'eosforce'
//ecc
const { ecc } = Eos.modules;
//get your public key from private key
const test_get_key_accounts = async () => {
    let publickKey = ecc.privateToPublic(privateKey)
    let result = Eos({ httpEndpoint: node_rpc_url}).getKeyAccounts({ public_key: publicKey });
    console.log( result );
}

test_get_key_accounts();
```

### getActions

get account transactions

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| accountName  | 用户名                                      |
| pos          | 起始位置                                     |
| offset       | 基于起始位置，偏移量                               |
| limit        | 最大筛选量                                    |

```javascript
const test_get_action = async () => {
    let result = await Eos({ httpEndpoint: node_rpc_url })
                    .getActions({
                            account_name: accountName, 
                            pos: pos, 
                            offset: offset, 
                            limit: 100 
                    });
    console.log(result);
}

test_get_action();

```

### getTransaction

get transaction details by transaction id 

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| id           | 订单ID                                     |

```javascript

const test_get_transaction = async () => {
    let result = Eos({ httpEndpoint: node_rpc_url }).getTransaction({ id: trx_id });
    console.log(result);
}

test_get_transaction();
```

### getTableRows


1. get account available
2. get account tokens
3. get votes table
4. get account votes table
5. get BP list


| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| scope        |                                          |
| code         |                                          |
| table        |                                          |


```javascript
    //get account available
    Eos({
        httpEndpoint: node_rpc_url 
    })
    .getTableRows({
      scope: 'eosio',
      code: 'eosio',
      table: 'accounts',
      table_key: accountName,
      limit: 10000,
      json: true,
    })
    .then(result => {
        console.log(result);
    });
    
    
    //get account tokens
    Eos({
        httpEndpoint: node_rpc_url 
    })
    .getTableRows({ 
        scope: accountName, 
        code: 'eosio.token', 
        table: 'accounts', 
        json: true, 
        limit: 1000 
    })
    .then(data => {
        console.log(data);
    });
    
    
    //get votes table
    Eos({
        httpEndpoint: node_rpc_url 
    })
    .getTableRows({ 
        scope: 'eosio', 
        code: 'eosio', 
        table: 'bps', 
        json: true, 
        limit: 1000 
    })
    .then(data => {
        console.log(data);    
    });
    
    
    //get account votes table
    Eos({
        httpEndpoint: node_rpc_url 
    })
    .getTableRows({ 
        scope: accountName, 
        code: 'eosio', 
        table: 'votes', 
        json: true, 
        limit: 1000 
    })
    .then(data => {
        console.log(data);
    });
    
    
    //get BP list
    Eos({ 
        httpEndpoint: node_rpc_url
    })
    .getTableRows({
        scope: 'eosio',
        code: 'eosio',
        table: 'schedules',
        table_key: schedule_version,
        json: true, 
        limit: 1000 
    })
    .then(result => {
        console.log(result);
    });
    //
```

### newaccount

newaccount

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| keyProvider  | 私钥                                       |
| chaninId     | 通过getInfo接口获得，返回数据里边的chanin_id           |

```javascript
const test_newaccount = async () => {
    let result = Eos({
                    httpEndpoint: node_rpc_url,
                    keyProvider: privateKey,
                    chainId
                })
                .newaccount('creator', 'need_created_name', publicKey, publicKey);
    console.log(result);
}

test_newaccount();
```

### transfer

transfer

| 参数           |                                          |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| keyProvider  | 私钥                                       |
| chainId      | 当前最高区块的chanin_id, 通过getBlock获得           |
| from         | 转账方，必须是当前私钥下边的账户                         |
| to           | 转账给谁                                     |
| amount       | 转账金额                                     |
| memo         | 转账说明                                     |

```javascript
let from = 'xxx', 
    to = 'apple2', 
    amount = 0.3, 
    memo = 'hi', 
    tokenSymbol = 'EOS', 
    precision = '4';
import BigNumber from 'bignumber.js';

const toBigNumber = asset => {
  if (BigNumber.isBigNumber(asset)) {
    return asset;
  } else if (isNaN(asset)) {
    if (!asset) return new BigNumber('0');
    const match = asset.match(/^([0-9.]+) EOS$/);
    const amount = match ? match[1] : '0';
    return new BigNumber(amount);
  } else {
    return new BigNumber(asset);
  }
};

const toAsset = (_amount, symbol = 'EOS', { precision = '4' } = {}) => {
  const amount = toBigNumber(_amount).toFixed(Number(precision));
  return [amount, symbol].join(' ');
};

const config ={
    httpEndpoint: node_rpc_url, //node url, http://192.168.82.173:8888
    keyProvider: privateKey, 
    chainId: 'chainId'
}

Eos(config)
.contract(tokenSymbol === 'EOS' ? 'eosio' : 'eosio.token')
.then(token => {
    return token.transfer(from, to, toAsset(amount, tokenSymbol, { precision }), memo);
});
```

### vote

vote

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| keyProvider  | 私钥                                       |
| chaninId     | 通过getInfo接口获得，返回数据里边的chanin_id           |



```javascript
// the formate is  'XXX EOS'
import BigNumber from 'bignumber.js'

const config ={
    httpEndpoint: node_rpc_url, //like http://192.168.82.173:8888
    keyProvider: privateKey, 
    chainId: 'chainId'
}

const toBigNumber = asset => {
  if (BigNumber.isBigNumber(asset)) {
    return asset;
  } else if (isNaN(asset)) {
    if (!asset) return new BigNumber('0');
    const match = asset.match(/^([0-9.]+) EOS$/);
    const amount = match ? match[1] : '0';
    return new BigNumber(amount);
  } else {
    return new BigNumber(asset);
  }
};

//
const toAsset = (_amount, symbol = 'EOS', { precision = '4' } = {}) => {
  const amount = toBigNumber(_amount).toFixed(Number(precision));
  return [amount, symbol].join(' ');
};

const test_vote = async () => {
   let result = await Eos(config).vote('xxx', 'eosforce', toAsset(1.9));
   console.log(result);
}

test_vote();
```

### unfreeze

unfreeze

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| keyProvider  | 私钥                                       |
| chaninId     | 通过getInfo接口获得，返回数据里边的chanin_id           |
| voter        | 投票人名称，拥有此私钥的账户中的一个                       |
| bpname       | 节点                                       |



```javascript
const config ={
    httpEndpoint: node_rpc_url, //节点rpc端口基础路径,如 http://192.168.82.173:8888
    keyProvider: privateKey, //私钥
    chainId: 'chainId'
}

const test_unfreeze = async () => {
   let result = await Eos(config).unfreeze(voter, bpname);    
   console.log(result);
}

test_unfreeze();
```


### claim

claim

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| keyProvider  | 私钥                                       |
| chaninId     | 通过getInfo接口获得，返回数据里边的chanin_id           |
| voter        | 投票人名称，拥有此私钥的账户中的一个                       |
| bpname       | 节点                                       |



```javascript
const config ={
    httpEndpoint: node_rpc_url, //节点rpc端口基础路径,如 http://192.168.82.173:8888
    keyProvider: privateKey, //私钥
    chainId: 'chainId'
}

const test_claim = async (voter, bpname) => {
    let result = await Eos(config).claim(voter, bpname);
    console.log(result);
}

test_claim();
```

### Get public key from private key

```javascript
import EOS from 'eosforce'
//从库中引入ecc
const { ecc } = Eos.modules;
//根据私钥算出公钥
let publickKey = ecc.privateToPublic(privateKey)
```

### Number to BiggerNumber

you can use this function let your number more precision

```javascript
import BigNumber from 'bignumber.js';

const toBigNumber = asset => {
  if (BigNumber.isBigNumber(asset)) {
    return asset;
  } else if (isNaN(asset)) {
    if (!asset) return new BigNumber('0');
    const match = asset.match(/^([0-9.]+) EOS$/);
    const amount = match ? match[1] : '0';
    return new BigNumber(amount);
  } else {
    return new BigNumber(asset);
  }
};

const toAsset = (_amount, symbol = 'EOS', { precision = '4' } = {}) => {
  const amount = toBigNumber(_amount).toFixed(Number(precision));
  return [amount, symbol].join(' ');
};
    
```
