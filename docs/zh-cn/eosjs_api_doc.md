# EOSJS API 文档

-----------------------------

## 安装

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

## 基础配置和需要的信息

```javascript
{
    httpEndpoint: node_rpc_url, //节点rpc端口基础路径,如 http://192.168.82.173:8888
    keyProvider: privateKey, //私钥,部分接口需要
    chainId: 'chainId'//部分接口需要, 通过getBlock获得
}
```

## 示例

接口都有可以运行的示例，可直接运行示例测试

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

## 接口列表

### getInfo

获取节点最新同步信息
参数:

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |

返回数据：

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

示例:
```javascript
 const test_get_info = async () => {
    let res = await  Eos(config).getInfo({});
    console.log(res);
 }
 
 test_get_info()

```
### getBlock

获取区块详细信息
传递参数:

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |

返回:

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

获取当前公钥下边有几个账户

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| public_key   | 根据私钥算出来的公钥                               |



```javascript
import EOS from 'eosforce'
//从库中引入ecc
const { ecc } = Eos.modules;
//根据私钥算出公钥
const test_get_key_accounts = async () => {
    let publickKey = ecc.privateToPublic(privateKey)
    let result = Eos({ httpEndpoint: node_rpc_url}).getKeyAccounts({ public_key: publicKey });
    console.log( result );
}

test_get_key_accounts();
```

### getActions

获取用户所有账单

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

获取账单详情

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

getTableRows涉及参数比较多，当前的功能有:

1. 查询用户可用余额
2. 获取用户token
3. 获取节点投票信息
4. 获取用户投票信息
5. 获取当届超级节点


| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| scope        |                                          |
| code         |                                          |
| table        |                                          |


```javascript
    //查询用户可用余额
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
    
    
    //获取用户token
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
    
    
    //获取节点投票信息
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
    
    
    //获取用户投票信息
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
    
    
    //获取当届超级节点
    Eos({ 
        httpEndpoint: node_rpc_url
    })
    .getTableRows({
        scope: 'eosio',
        code: 'eosio',
        table: 'schedules',
        table_key: schedule_version,//查看getBlock接口,有获得当前参数的方式
        json: true, 
        limit: 1000 
    })
    .then(result => {
        console.log(result);
    });
    //
```

### newaccount

创建账户

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

转账

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
    memo = '你好', 
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
    httpEndpoint: node_rpc_url, //节点rpc端口基础路径,如 http://192.168.82.173:8888
    keyProvider: privateKey, //私钥
    chainId: 'chainId'
}

Eos(config)
.contract(tokenSymbol === 'EOS' ? 'eosio' : 'eosio.token')
.then(token => {
    return token.transfer(from, to, toAsset(amount, tokenSymbol, { precision }), memo);
});
```

### vote

投票

| 参数           | 说明                                       |
| ------------ | ---------------------------------------- |
| httpEndpoint | 节点rpc端口基础路径,如 http://192.168.82.173:8888 |
| keyProvider  | 私钥                                       |
| chaninId     | 通过getInfo接口获得，返回数据里边的chanin_id           |



```javascript
// '字符串或数字或 bignumber 格式转化为 XXX EOS 格式'
import BigNumber from 'bignumber.js'

const config ={
    httpEndpoint: node_rpc_url, //节点rpc端口基础路径,如 http://192.168.82.173:8888
    keyProvider: privateKey, //私钥
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

//为了数据准确性，toAsset函数是必要的，可以加入自己的公共函数
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

领取分红

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

解冻

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

### 私钥转公钥函数

```javascript
import EOS from 'eosforce'
//从库中引入ecc
const { ecc } = Eos.modules;
//根据私钥算出公钥
let publickKey = ecc.privateToPublic(privateKey)
```

### 输入数字转为接口需要的格式

接口中用到的toAsset函数，都可以用此函数

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
