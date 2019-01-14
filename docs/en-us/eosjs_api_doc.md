# EOSJS API 文档

-----------------------------

## Install

if you are using npm 
```bash
npm install eosforce
npm install bignumber.js 
```
if you are using yarn
```bash
yarn add eosforce
yarn add bignumber.js 
```

## Base Config

```javascript
{
    httpEndpoint: node_rpc_url, //like http://192.168.82.173:8888
    keyProvider: privateKey, 
    chainId: 'chainId'//you can get chain Id though getInfo
}
```

## Example


```javascript
import Eos from 'eosforce';
const config = { 
    httpEndpoint: node_rpc_url
}
//get node info
const test_get_info = async () => {
    let res = await  Eos(config).getInfo({});
    console.log(res);
}

test_get_info()
```

## Api

### getInfo

get node info

| params           | des                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | like http://192.168.82.173:8888 |

return：

| params                          | des         |
| --------------------------- | ---------- |
| block_cpu_limit             |            |
| block_net_limit             |            |
| chain_id                    |            |
| head_block_id               | current block id |
| head_block_num              | current block block_num |
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

| params           | des                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint |  |

return:

| key                 | des                                           |
| ------------------ | -------------------------------------------- |
| action_mroot       |                                              |
| block_extensions   |                                              |
| block_num          |                                    |
| confirmed          |                                              |
| header_extensions  |                                              |
| id                 |                                              |
| new_producers      |                                              |
| previous           |                                              |
| producer           |                                          |
| producer_signature |                                              |
| ref_block_prefix   |                                              |
| schedule_version   |  |
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

get accounts by key

| parmas       | des                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | like http://192.168.82.173:8888 |
| public_key   | you can get it from private key                             |



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

| params           | des                                      |
| ------------ | ---------------------------------------- |
| httpEndpoint |                                        |
| accountName  |                                       |
| pos          |                                      |
| offset       |                            |
| limit        |                                   |

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

| params           | des                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | like http://192.168.82.173:8888 |
| id           | transaction id                                     |

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
5. get BP list (BP is short for Block Producer, EOSForce have 23 Block Product)


| params           | des                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | like http://192.168.82.173:8888 |
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

| params           | des                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | like http://192.168.82.173:8888 |
| keyProvider  | private key                                      |
| chaninId     | you can get it though getInfo        |

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

| params           | des                                    |
| ------------ | ---------------------------------------- |
| httpEndpoint | like http://192.168.82.173:8888 |
| keyProvider  | private key                                       |
| chainId      | get it though getInfo           |
| from         |                          |
| to           |                                    |
| amount       |                                   |
| memo         |                               |

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

| params           | des                                      |
| ------------ | ---------------------------------------- |
| httpEndpoint | like http://192.168.82.173:8888 |
| keyProvider  | private key                                       |
| chaninId     | you can get it though getInfo          |



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

| params           | des                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint |  |
| keyProvider  | private key                                       |
| chaninId     | get this though getInfo API           |
| voter        |                        |
| bpname       |                                       |



```javascript
const config ={
    httpEndpoint: node_rpc_url, 
    keyProvider: privateKey,
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

| params           | des                                     |
| ------------ | ---------------------------------------- |
| httpEndpoint | like http://192.168.82.173:8888 |
| keyProvider  | private key                                       |
| chaninId     | you can get this though getInfo          |
| voter        |                        |
| bpname       |                                        |



```javascript
const config ={
    httpEndpoint: node_rpc_url, //like http://192.168.82.173:8888
    keyProvider: privateKey, 
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
const { ecc } = Eos.modules;
//get public key from private key
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
