# EOSForce测试主网搭建说明

----------------------------------------------------

在很多时候为了方便开发测试，需要自己单独启动一套主网，这里是一个启动主网的说明，也可以作为超级节点运维的参考。

对应eosforce提交： 0e10da90619e05296f166b11e5a7536c5fa78e71

TODO : 添加使用config.ini启动说明
TODO : 添加docker启动说明
TODO : 添加多机器启动说明

## 0. eosforce 与 eos.io官方版本的区别

eosforce为了保证链的安全性，关闭了很多官方版本中未经验证的功能，
eosforce中只添加了最小需求的系统用户和合约，这些用户和合约是内置在代码中的，在节点初始化区块链时加载的，
这样eosforce启动时不需要向官方版本中的bios流程中有大量的创建用户和加载系统合约的步骤。

eosforce在bios时会指定genesis.json中配置的23个初始超级节点，这和eos.io官方版本不同， 所以不需要注册超级节点步骤，
开始启动时，直接启动23个对应的节点即可。

## 1. eosforce bios基本步骤

- 1 编译eosforce
- 2 通过program/genesis工具生成23个节点的公私钥配置和对应的genesis.json
- 3 启动23个超级节点（也可以少于23个）

步骤要比官方启动少很多，需要注意的就是保证配置正确即可。

**注意** 这里示例中使用的是ubuntu 18.04版本，需要 python 3.6.

## 2. 配置说明

nodeos 可以通过指定config.ini配置启动信息， 也可以通过命令行参数指定，
一般来说配置文件比较适合生产环境或者容器启动，如果是测试用的节点可以使用命令行参数，便于脚本启动.

这里我们使用命令行参数启动nodeos，便于脚本启动.

nodeos 参数如下：

| 名字                     | 说明                                                           | 示例值                                                                    | 对应config.ini的值     |
|:-------------------------|:---------------------------------------------------------------|:--------------------------------------------------------------------------|:-----------------------|
| --blocks-dir             | 存放blocks文件的目录，不同节点不能重复                         | "./nodes/01-biosbpa/blocks"                                               | blocks-dir             |
| --config-dir             | 配置文件及合约目录，用命令行启动可以用相同的                   | "./config"                                                                |                        |
| --data-dir               | 存放data文件目录，不同节点不能重复                             | "./nodes/01-biosbpa/data"                                                 |                        |
| --http-server-address    | http server 地址                                               | 127.0.0.1:8001                                                            | http-server-address    |
| --p2p-listen-endpoint    | p2p 监听地址                                                   | 127.0.0.1:9001                                                            | p2p-listen-endpoint    |
| --max-clients            | 最大客户端数量                                                 | 64                                                                        | max-clients            |
| --p2p-max-nodes-per-host | 单个ip上节点最大值，如果部署在单台机器的话需要配置             | 64                                                                        | p2p-max-nodes-per-host |
| --signature-provider     | 超级节点公私钥，格式是public-key=KEY:private-key               |                                                                           | signature-provider     |
| --plugin                 | 要启动的插件                                                   | 一般是  eosio::http_plugin eosio::chain_api_plugin eosio::producer_plugin | plugin                 |
| --p2p-peer-address       | p2p对端地址                                                    | 127.0.0.1:9002, 就是其他节点对应的p2p监听地址                             |                        |
| --producer-name          | 超级节点名字，需要注意的是启动时一定是对应genesis.json中的配置 | biospba                                                                   | producer-name          |

这里要特别注意的是 --config-dir
eosforce要求这个目录下包含以下文件：

```
config
├── System.abi
├── System.wasm
├── eosio.token.abi
├── eosio.token.wasm
├── config.ini
└── genesis.json
```

因为我们使用命令行参数，所以config.ini留一个空文件即可

以下是个例子：

```bash
./nodeos \
    --blocks-dir ./nodes/20-biosbpt/blocks \
    --config-dir ./nodes/20-biosbpt/../../config \
    --data-dir ./nodes/20-biosbpt \
    --http-server-address 127.0.0.1:8020 \
    --p2p-listen-endpoint 127.0.0.1:9020 \
    --max-clients 33 \
    --p2p-max-nodes-per-host 33 \
    --enable-stale-production \
    --producer-name biosbpt \
    --signature-provider=EOS5bijTJg8ZkGNTuL3VfMcMc3bYdLtZ1faJJPYHRB1ShDZLtkeRY=KEY:5Jgau5mVjCiGrTGrvLr9YcyWfWdWWTrPYVwxUhgaRc5ixKWCCff \
    --plugin eosio::http_plugin \
    --plugin eosio::chain_api_plugin \
    --plugin eosio::producer_plugin    \
    --p2p-peer-address localhost:9001
```

**注意** 启动测试节点的时候，我们可以不是特别的注意安全问题，所以这里启动plugin都是全节点，生产环境中的超级节点需要注意安全问题。

这么长的命令肯定不适合手动输入，可以写脚本来启动，最后我们使用python做一个启动脚本。

## 3. 启动步骤

### 3.1 编译eosforce

获取代码：

```bash
git clone https://github.com/eosforce/eosforce.git
```

编译：

```bash
cd eosforce
 ./eosio_build.sh
```

注意脚本执行过程中会有几次确认，注意通过。

成功会有以下提示：

```bash
	(  ____ \(  ___  )(  ____ \\__   __/(  ___  )
	| (    \/| (   ) || (    \/   ) (   | (   ) |
	| (__    | |   | || (_____    | |   | |   | |
	|  __)   | |   | |(_____  )   | |   | |   | |
	| (      | |   | |      ) |   | |   | |   | |
	| (____/\| (___) |/\____) |___) (___| (___) |
	(_______/(_______)\_______)\_______/(_______)

	EOSIO has been successfully built. 00:12:35

	To verify your installation run the following commands:

	export PATH=${HOME}/opt/mongodb/bin:$PATH
	/home/fy/opt/mongodb/bin/mongod -f /home/fy/opt/mongodb/mongod.conf &
	cd /home/fy/projects/eosforce_gen/build; make test

	For more information:
	EOSIO website: https://eos.io
	EOSIO Telegram channel @ https://t.me/EOSProject
	EOSIO resources: https://eos.io/resources/
	EOSIO Stack Exchange: https://eosio.stackexchange.com
	EOSIO wiki: https://github.com/EOSIO/eos/wiki

```

这里要注意的是，如果要执行test，最好确认关闭所有 nodeos 进程，否则会引起冲突

编译结果默认的位置是 eosforce/build

为了不污染全局环境而引起一些错误，不建议使用 make install。

### 3.2 配置

为了方便启动，这里把节点工作目录设置为 eosforce/build/testrun/,
相应的配置文件目录为 eosforce/build/testrun/config/

```bash
cd ./build
mkdir testrun
```

首先生成 genesis.json
这里eosforce提供了一个工具，便于随机生成所有超级节点的key

```bash
cd testrun
../programs/genesis/genesis
```

这时目录下会生成以下文件：

- config.ini 生成的超级节点key，便于复制到config.ini中
- genesis.json 生成的创始配置，我们后续就使用这个配置
- key.json 对应超级节点账户公钥的私钥
- sigkey.json 超级节点签名用的公钥对应的私钥，后面会使用这个来启动nodeos

这里注意区分超级节点 *账户key* 和 *sign_key*, 启动超级节点需要使用sign_key作为signature-provider配置。

对于genesis.json，我们直接移动进config目录下：

```bash
mkdir ./config
mv ./genesis.json ./config/
```

复制需要的合约文件进入config，同时创建一个空的config.ini,
**注意** ： eosforce所使用的合约是 eosforce/contracts/System 和 eosforce/contracts/eosio.token,
**不是** **不是** **不是**  ~~eosforce/contracts/eosio.system~~

```bash
cp ../contracts/System/System.abi ./config/
cp ../contracts/System/System.wasm ./config/
cp ../contracts/System01/System01.abi ./config/
cp ../contracts/System01/System01.wasm ./config/
cp ../contracts/eosio.token/eosio.token.abi ./config/
cp ../contracts/eosio.token/eosio.token.wasm ./config/
cp ../contracts/eosio.msig/eosio.msig.abi ./config/
cp ../contracts/eosio.msig/eosio.msig.wasm ./config/
cp ../contracts/eosio.bios/eosio.bios.abi ./config/
cp ../contracts/eosio.bios/eosio.bios.wasm ./config/
echo "" > ./config/config.ini
```

这时config下有如下文件

```
config.ini  eosio.token.abi  eosio.token.wasm  genesis.json  System.abi  System.wasm
```

### 3.3 启动超级节点

完成配置之后可以启动超级节点，这里我们启动三个节点,
由于第0届超级节点是由genesis.json中指定的，我们启动时需要对应genesis.json中的sign_key启动超级节点，
这里我们启动 biosbpa biosbpb biosbpc三个节点。

在genesis.json中配置如下：

```json
{
  "initial_timestamp": "2018-05-28T12:00:00.000",
  "initial_key": "EOS1111111111111111111111111111111114T1Anm",
        ...
  "initial_configuration": { ... },
  "initial_account_list": [{
      "key": "EOS5XLWw7xu6qR1yqCCYqxYEMggCyKPWUTDyAjA1pgECFtcd39G41",
      "asset": "1.0000 EOS",
      "name": "biosbpa"
    }],

  "initial_producer_list": [{
      "name": "biosbpa",
      "bpkey": "EOS63o5TEdLsQrxFTUobiDupHQMX8K5pEs7PPk91afmpkrkiprokc",
      "commission_rate": 0,
      "url": ""
    }]
}
```

initial_account_list中是初始账号的列表，key是账号公钥，对应之前生成的key.json中的私钥
initial_producer_list是超级节点的列表，bpkey对应的私钥在之前生成的sigkey.json中

需要注意的是，eosforce启动之后会处理b1账户，所以为了防止一直报错，需要在initial_account_list中添加一个name为b1的账户。
否则会报以下错误：

```log
2861606ms thread-0   controller.cpp:875            push_transaction     ] -------call onfee function tx
2862013ms thread-0   wasm_interface.cpp:933        eosio_assert         ] message: b1 is not found in accounts table 
2862013ms thread-0   controller.cpp:888            push_transaction     ] ---trnasction exe failed--------trace: {"id":"13a4c6b68425acb4fa4aae0d2e92fcb557939fa1f4b2e46228025bc8560b597f","elapsed":0,"net_usage":0,"scheduled":false,"action_traces":[{"receipt":{"receiver":"","act_digest":"0000000000000000000000000000000000000000000000000000000000000000","global_sequence":0,"recv_sequence":0,"auth_sequence":[],"code_sequence":0,"abi_sequence":0},"act":{"account":"","name":"","authorization":[],"data":""},"elapsed":0,"cpu_usage":0,"console":"","total_cpu_usage":0,"trx_id":"0000000000000000000000000000000000000000000000000000000000000000","inline_traces":[]}],"failed_dtrx_trace":null,"except":{"code":3050003,"name":"eosio_assert_message_exception","message":"eosio_assert_message assertion failure","stack":[{"context":{"level":"error","file":"wasm_interface.cpp","line":934,"method":"eosio_assert","hostname":"","thread_name":"thread-0","timestamp":"2018-07-13T03:47:42.013"},"format":"assertion failure with message: ${s}","data":{"s":"b1 is not found in accounts table"}},{"context":{"level":"warn","file":"apply_context.cpp","line":60,"method":"exec_one","hostname":"","thread_name":"thread-0","timestamp":"2018-07-13T03:47:42.013"},"format":"","data":{"_pending_console_output.str()":""}}]}}
2862014ms thread-0   controller.cpp:875            push_transaction     ] -------call onfee function tx

```

也可以启动主网之后手动创建一个b1账户

这里我们启动超级节点需要对应超级节点的name，bpkey和在sigkey.json中的sig_key,
分别在下面的{name},{bpkey},{sig_key}

```bash
./nodeos --blocks-dir ./nodes/biosbpa/blocks --config-dir ./config --data-dir ./nodes/biosbpa --http-server-address 127.0.0.1:8021 --p2p-listen-endpoint 127.0.0.1:9021 --max-clients 64 --p2p-max-nodes-per-host 64 --enable-stale-production --producer-name biosbpa --signature-provider={biosbpa-bpkey}=KEY:{biosbpa-sig_key} --plugin eosio::http_plugin --plugin eosio::chain_api_plugin --plugin eosio::producer_plugin

./nodeos --blocks-dir ./nodes/biosbpb/blocks --config-dir ./config --data-dir ./nodes/biosbpb --http-server-address 127.0.0.1:8022 --p2p-listen-endpoint 127.0.0.1:9022 --max-clients 64 --p2p-max-nodes-per-host 64 --enable-stale-production --producer-name biosbpb --signature-provider={biosbpb-bpkey}=KEY:{biosbpb-sig_key} --plugin eosio::http_plugin --plugin eosio::chain_api_plugin --plugin eosio::producer_plugin --p2p-peer-address localhost:9021

./nodeos --blocks-dir ./nodes/biosbpc/blocks --config-dir ./config --data-dir ./nodes/biosbpc --http-server-address 127.0.0.1:8023 --p2p-listen-endpoint 127.0.0.1:9023 --max-clients 64 --p2p-max-nodes-per-host 64 --enable-stale-production --producer-name biosbpc --signature-provider={biosbpc-bpkey}=KEY:{biosbpc-sig_key} --plugin eosio::http_plugin --plugin eosio::chain_api_plugin --plugin eosio::producer_plugin --p2p-peer-address localhost:9021 --p2p-peer-address localhost:9022

```

注意其中的 --p2p-peer-address 依照启动顺序监听已经启动的节点

如果启动成功，可以看到节点出块日志

需要注意的是，EOS中如果要出块的节点没有启动，则这一块会轮空，依然会等待出块的间隔，直到能出块的节点出块，
eosforce代码中默认出块间隔是3s，这意味着a，b，c三个节点各自出块之后，会等待 20 * 3 = 60s之后，a节点才会出下一个块

### 3.4 验证执行合约

参见 [https://github.com/eosforce/contracts/tree/master/System#command-reference](https://github.com/eosforce/contracts/tree/master/System#command-reference)
连接之前设置的http-server-address 127.0.0.1:8022，例如使用如下执行：

```bash
./cleos -u http://127.0.0.1:8022 get table eosio eosio bps
```

获取超级节点列表

### 3.5 使用客户端钱包连接测试网

TODO

## 4. 通过python脚本启动

这个脚本改写自 eosforce/tutorials/bios-boot-tutorial/bios-boot-tutorial.py
可以方便的在单机上一键启动23个节点构成测试网。

### 4.1 脚本

```python
#!/usr/bin/env python3

import argparse
import json
import os
import re
import subprocess
import sys
import time

args = None
logFile = None

unlockTimeout = 999999999

def jsonArg(a):
    return " '" + json.dumps(a) + "' "

def run(args):
    print('bios-boot-tutorial.py:', args)
    logFile.write(args + '\n')
    if subprocess.call(args, shell=True):
        print('bios-boot-tutorial.py: exiting because of error')
        sys.exit(1)

def retry(args):
    while True:
        print('bios-boot-tutorial.py:', args)
        logFile.write(args + '\n')
        if subprocess.call(args, shell=True):
            print('*** Retry')
        else:
            break

def background(args):
    print('bios-boot-tutorial.py:', args)
    logFile.write(args + '\n')
    return subprocess.Popen(args, shell=True)

def sleep(t):
    print('sleep', t, '...')
    time.sleep(t)
    print('resume')

def makeGenesis():
    run('rm -rf ' + os.path.abspath(args.config_dir))
    run('mkdir -p ' + os.path.abspath(args.config_dir))
    run('mkdir -p ' + os.path.abspath(args.config_dir) + '/keys/' )

    run('cp ' + os.path.abspath(args.contracts_dir) + '/eosio.token/eosio.token.abi ' + os.path.abspath(args.config_dir))
    run('cp ' + os.path.abspath(args.contracts_dir) + '/eosio.token/eosio.token.wasm ' + os.path.abspath(args.config_dir))
    run('cp ' + os.path.abspath(args.contracts_dir) + '/System/System.abi ' + os.path.abspath(args.config_dir))
    run('cp ' + os.path.abspath(args.contracts_dir) + '/System/System.wasm ' + os.path.abspath(args.config_dir))

    run('echo "" > ' + os.path.abspath(args.config_dir) + '/config.ini')

    run('../programs/genesis/genesis')
    run('mv ./genesis.json ' + os.path.abspath(args.config_dir))

    run('mv ./key.json ' + os.path.abspath(args.config_dir) + '/keys/')
    run('mv ./sigkey.json ' + os.path.abspath(args.config_dir) + '/keys/')

def addB1Account():
    run(args.cleos + 'create account eosforce b1 ' + initAccounts[len(initAccounts) - 1]['key'])

def startWallet():
    run('rm -rf ' + os.path.abspath(args.wallet_dir))
    run('mkdir -p ' + os.path.abspath(args.wallet_dir))
    background(args.keosd + ' --unlock-timeout %d --http-server-address 127.0.0.1:6666 --wallet-dir %s' % (unlockTimeout, os.path.abspath(args.wallet_dir)))
    sleep(.4)
    run(args.cleos + 'wallet create')

def importKeys():
    keys = {}
    for a in initAccountsKeys:
        key = a[1]
        if not key in keys:
            keys[key] = True
            # note : new develop eosforce change this command to wallet import --key pk
            # so need change this cmd
            run(args.cleos + 'wallet import key ' + key)

def startNode(nodeIndex, bpaccount, key):
    dir = args.nodes_dir + ('%02d-' % nodeIndex) + bpaccount['name'] + '/'
    run('rm -rf ' + dir)
    run('mkdir -p ' + dir)
    otherOpts = ''.join(list(map(lambda i: '    --p2p-peer-address localhost:' + str(9001 + i), range(nodeIndex - 1))))
    if not nodeIndex: otherOpts += (
        '    --plugin eosio::history_plugin'
        '    --plugin eosio::history_api_plugin'
    )

    print('bpaccount ', bpaccount)
    print('key ', key, ' ', key[1])

    cmd = (
        args.nodeos +
        '    --blocks-dir ' + os.path.abspath(dir) + '/blocks'
        '    --config-dir ' + os.path.abspath(dir) + '/../../config'
        '    --data-dir ' + os.path.abspath(dir) +
        '    --http-server-address 127.0.0.1:' + str(8000 + nodeIndex) +
        '    --p2p-listen-endpoint 127.0.0.1:' + str(9000 + nodeIndex) +
        '    --max-clients ' + str(maxClients) +
        '    --p2p-max-nodes-per-host ' + str(maxClients) +
        '    --enable-stale-production'
        '    --producer-name ' + bpaccount['name'] +
        '    --signature-provider=' + bpaccount['bpkey'] + '=KEY:' + key[1] +
        '    --plugin eosio::http_plugin'
        '    --plugin eosio::chain_api_plugin'
        '    --plugin eosio::producer_plugin' +
        otherOpts)
    with open(dir + '../' + bpaccount['name'] + '.log', mode='w') as f:
        f.write(cmd + '\n\n')
    background(cmd + '    2>>' + dir + '../' + bpaccount['name'] + '.log')

def startProducers(inits, keys):
    for i in range(0, len(inits)):
        startNode(i + 1, initProducers[i], keys[i])

def listProducers():
    run(args.cleos + 'get table eosio eosio bps')

def stepKillAll():
    run('killall keosd nodeos || true')
    sleep(1.5)

def stepStartWallet():
    startWallet()
    importKeys()

def stepStartProducers():
    startProducers(initProducers, initProducerSigKeys)
    sleep(1)
    addB1Account()
    sleep(args.producer_sync_delay)

def stepLog():
    run('tail -n 1000 ' + args.nodes_dir + 'biosbpa.log')
    listProducers()

# Command Line Arguments
parser = argparse.ArgumentParser()

commands = [
    ('k', 'kill',           stepKillAll,                True,    "Kill all nodeos and keosd processes"),
    ('w', 'wallet',         stepStartWallet,            True,    "Start keosd, create wallet, fill with keys"),
    ('P', 'start-prod',     stepStartProducers,         True,    "Start producers"),
    ('l', 'log',            stepLog,                    True,    "Show tail of node's log"),
]

parser.add_argument('--contracts-dir', metavar='', help="Path to contracts directory", default='../contracts/')
parser.add_argument('--cleos', metavar='', help="Cleos command", default='../programs/cleos/cleos --wallet-url http://localhost:6666 ')
parser.add_argument('--nodeos', metavar='', help="Path to nodeos binary", default='../programs/nodeos/nodeos')
parser.add_argument('--nodes-dir', metavar='', help="Path to nodes directory", default='./nodes/')
parser.add_argument('--keosd', metavar='', help="Path to keosd binary", default='../programs/keosd/keosd')
parser.add_argument('--log-path', metavar='', help="Path to log file", default='./output.log')
parser.add_argument('--wallet-dir', metavar='', help="Path to wallet directory", default='./wallet/')
parser.add_argument('--config-dir', metavar='', help="Path to config directory", default='./config')
parser.add_argument('--producer-sync-delay', metavar='', help="Time (s) to sleep to allow producers to sync", type=int, default=10)
parser.add_argument('-a', '--all', action='store_true', help="Do everything marked with (*)")
parser.add_argument('-H', '--http-port', type=int, default=8001, metavar='', help='HTTP port for cleos')

for (flag, command, function, inAll, help) in commands:
    prefix = ''
    if inAll: prefix += '*'
    if prefix: help = '(' + prefix + ') ' + help
    if flag:
        parser.add_argument('-' + flag, '--' + command, action='store_true', help=help, dest=command)
    else:
        parser.add_argument('--' + command, action='store_true', help=help, dest=command)

args = parser.parse_args()

args.cleos += '--url http://localhost:%d ' % args.http_port

logFile = open(args.log_path, 'a')

logFile.write('\n\n' + '*' * 80 + '\n\n\n')

makeGenesis()

with open(os.path.abspath(args.config_dir) + '/genesis.json') as f:
    a = json.load(f)
    initAccounts = a['initial_account_list']
    initProducers = a['initial_producer_list']

with open(os.path.abspath(args.config_dir) + '/keys/sigkey.json') as f:
    a = json.load(f)
    initProducerSigKeys = a['keymap']

with open(os.path.abspath(args.config_dir) + '/keys/key.json') as f:
    a = json.load(f)
    initAccountsKeys = a['keymap']

maxClients = len(initProducers) + 10

haveCommand = False
for (flag, command, function, inAll, help) in commands:
    if getattr(args, command) or inAll and args.all:
        if function:
            haveCommand = True
            function()

if not haveCommand:
    print('bios-boot-tutorial.py: Tell me what to do. -a does almost everything. -h shows options.')
```

### 4.2 执行说明

py脚本默认的执行位置在 eosforce/build/XXX 目录下， ‘XXX’可以任意指定

执行即可：

```bash
chmod u+x ./bios_boot_eosforce.py
./bios_boot_eosforce.py
```

成功后会查询所有当前bp，各个节点日志在 eosforce/build/XXX/nodes/ 目录下