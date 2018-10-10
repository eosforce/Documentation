# 智能合约起步

第一步：安装合约开发工具集（CDT）

EOSFORCEIO 合约开发工具集 - 按照安装指导继续。eosiocpp工具包含在工具集中，该工具编译编译合约并且生成 ABI 文件。

首先克隆代码

	git clone --recursive https://github.com/eosio/eosio.cdt
	cd eosio.cdt

然后运行 build.sh 并且提供打算部署 EOSFORCEIO 区块链的核心符号。build.sh 将自动安装所需要的依赖。

	./build.sh <CORE_SYMBOL>

最后，安装构建，安装将把核心安装到 /usr/local/eosio.cdt， 顶层工具（编译器，链接器等等）的符号链接安装到 /usr/local/bin


	$ sudo ./install.sh


第二步：启动节点

如果使用 docker 并且 容器不在运行，执行下面命令。

	docker start eosio

如果在本地运行 nodeos , 用下列单个命令可以启动单节点区块链。

	$ nodeos -e -p eosio --plugin eosio::chain_api_plugin \
        --plugin eosio::history_api_plugin

这个命令设置了许多标志并且装载了本教程其余部分所需要的可选插件。假如一切顺利，妹0.5秒你将看到一条区块生成消息。

在 docker 上，

	docker logs --tail 25 eosio

	...
	3165501ms thread-0   producer_plugin.cpp:944       produce_block        ] Produced block 00000a4c898956e0... #2636 @ 2018-05-25T16:52:45.500 signed by eosio [trxs: 0, lib: 2635, confirmed: 0]
	3166004ms thread-0   producer_plugin.cpp:944       produce_block        ] Produced block 00000a4d2d4a5893... #2637 @ 2018-05-25T16:52:46.000 signed by eosio [trxs: 0, lib: 2636, confirmed: 0]
	...

第三步：创建钱包

钱包是私钥仓库，必须用私钥来认证区块链上 actions。这些私钥加密存储在磁盘上，使用密码来保护。密码应该存储在一个安全的密码管理器中或者记下来。

	$ cleos wallet create --to-console
	Creating wallet: default
	Save password to use in the future to unlock this wallet.
	Without password imported keys will not be retrievable.
	"PW5JuBXoXJ8JHiCTXf...."


钱包经过一段时间将自动锁定，通过下列命令解锁

	$ cleos wallet unlock
	password:

为了安全目的，当不适用钱包的时候，一般最后让其锁定。为了锁定钱包而不关闭 nodeos , 可以执行下列命令

	cleos wallet lock
	Locked: default

本文其余部分需要钱包处于解锁状态。


装载教程 Key

上述步骤发起的私链伴随创建了一堆初始秘钥，必须导入钱包（参见下文）

	$ cleos wallet import --private-key 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
	imported private key for: EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV


第四部：装载 BIOS 合约

现在我们有一个钱包已经装载了 eosio 账户的私钥，我们部署一个缺省的系统合约。为了开发的目的，我们使用缺省的 eosio.bios 合约。
这个合约使你能直接控制其它账户的资源分配并且能访问其它有特权的 API 调用。在公链上，这个合约将管理 tokens 抵押及解押以为合约申请CPU、网络、内存。

eosio.bios 合约在 EOSFORCEIO 源代码 contracts/eosio.bios 目录下。下面的命令序列假设从 EOSFORCEIO 源代码根目录执行，但是你可以指定完整路径
${EOSIO_SOURCE}/build/contracts/eosio.bios 从任何地方执行。

如果你用 docker, 命令是：

	$ cleos set contract eosio contracts/eosio.bios -p eosio@active
	Reading WAST...
	Assembling WASM...
	Publishing contract...
	executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
	#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
	#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...

如果从源代码构建，命令是：

	$ cleos set contract eosio build/contracts/eosio.bios -p eosio@active
	Reading WAST...
	Assembling WASM...
	Publishing contract...
	executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
	#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
	#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...

命令序列的结果是 cleos 生成包含两个 actions 的 transaction, eosio::setcode 和 eosio::setabi。

code 定义了合约怎样运行， abi 描述了参数怎样在二进制和 json 表示间转换。虽然 abi 技术上是可选的， DOSFORCEIO 所有工具都依赖它为了方便使用。

任何时候执行一个 transaction ,将看到如下输出：

	executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083  4068 bytes  10000 cycles
	#         eosio <= eosio::setcode               {"account":"eosio","vmtype":0,"vmversion":0,"code":"0061736d0100000001ab011960037f7e7f0060057f7e7e7e...
	#         eosio <= eosio::setabi                {"account":"eosio","abi":{"types":[],"structs":[{"name":"set_account_limits","base":"","fields":[{"n...

可以这样解读：eosio 定义的 action setcode 被 eosio 合约用参数 {args...} 执行。

	#         ${executor} <= ${contract}:${action} ${args...}
	> console output from this execution, if any

后面我们将会看到 actions 可以被多个合约处理。

这个调用的最后一个参数是 -p eosio@active 。 这告诉 cleos 用 eosio 账户的 active 授权来签署 这个 action ，比如说用之前为 eosio 账户导入的私钥
来签署这个 action 。

第五步：创建账户

现在我们已经设置了基本的系统合约，我们可以开始创建我们自己的账户。我们将创建两个账户 user 和 tester, 我们需要为每个账户关联一个私钥。
这个例子中，两个账户将使用同一个私钥。

我们首先为账户生成私钥。

	$ cleos create key --to-console
	Private key: 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
	Public key: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4

然后我们将使用导入钱包：

	$ cleos wallet import --private-key 5Jmsawgsp1tQ3GD6JyGCwy1dcvqKZgX6ugMVMdjirx85iv5VyPR
	imported private key for: EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4

主要用 cleos 实际生成的私钥，不要用例子中显示的。

私钥不会自动加入钱包，因此忽略这步将导致失去对账户的控制。

创建两个用户账户

接下来我们创建两个账户， user 和 tester, 用上面创建和导入的私钥。

	$ cleos create account eosio user EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
	executed transaction: 8aedb926cc1ca31642ada8daf4350833c95cbe98b869230f44da76d70f6d6242  364 bytes  1000 cycles
	#         eosio <= eosio::newaccount            {"creator":"eosio","name":"user","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...
	
	$ cleos create account eosio tester EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
	executed transaction: 414cf0dc7740d22474992779b2416b0eabdbc91522c16521307dd682051af083 366 bytes  1000 cycles
	#         eosio <= eosio::newaccount            {"creator":"eosio","name":"tester","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxentZZ...

注意：create account 命令需要两个密钥，一个是 OwnerKey (在生产环境上应该高度保密)，另一个是 ActiveKey。这个例子中，使用了同一把密钥。

因为我们使用了 eosio::history_api_plugin ， 因此我们可以查询我们的密钥控制的所有账户：

	$ cleos get accounts EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
	{
	  "account_names": [
	    "tester",
	    "user"
	  ]
	}

# EOSFORCEIO Token 合约介绍

Eosio.token, Exchange, 及 Eosio.msig 合约

在这个阶段区块链并不能做很多，因此让我们部署 eosio.token 合约。这个合约使能创建许多不同的 tokens, 都运行在相同的合约上但是潜在被不同的用户管理。

在我们部署 token 合约前，先要将要部署上去的账户。

	$ cleos create account eosio eosio.token \
        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 \
	...

然诺我们可以部署合约，合约在 ${EOSIO_SOURCE}/build/contracts/eosio.token 目录下。

	$ cleos set contract eosio.token build/contracts/eosio.token -p eosio.token@active
	Reading WAST...
	Assembling WASM...
	Publishing contract...
	executed transaction: 528bdbce1181dc5fd72a24e4181e6587dace8ab43b2d7ac9b22b2017992a07ad  8708 bytes  10000 cycles
	#         eosio <= eosio::setcode               {"account":"eosio.token","vmtype":0,"vmversion":0,"code":"0061736d0100000001ce011d60067f7e7f7f7f7f00...
	#         eosio <= eosio::setabi                {"account":"eosio.token","abi":{"types":[],"structs":[{"name":"transfer","base":"","fields":[{"name"...

创建 Currency Token

可以查看 eosio.token 合约的接口，定义在 contracts/eosio.token/eosio.token.hpp:

	void create( account_name issuer,
                asset        maximum_supply );


   	void issue( account_name to, asset quantity, string memo );

   	void transfer( account_name from,
                  account_name to,
                  asset        quantity,
                  string       memo );

为了创建新的 token, 我们必须调用 create(...) action 用合适的参数。这个命令将使用最大供给的符号来识别这个 token 以区别于其它 tokens。
发行人有权调用发行与或执行其它动作比如冻结，撤销，及白名单拥有者。

调用这个命令的简洁方法，用位置参数：

	$ cleos push action eosio.token create '[ "eosio", "1000000000.0000 SYS"]' \
	         -p eosio.token@active
	executed transaction: 0e49a421f6e75f4c5e09dd738a02d3f51bd18a0cf31894f68d335cd70d9c0e12  120 bytes  1000 cycles
	#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 SYS"}

可选的调用这个命令的更详细的方式，使用命名参数：

	$ cleos push action eosio.token create \
	        '{"issuer":"eosio", "maximum_supply":"1000000000.0000 SYS"}' \
	        -p eosio.token@active
	executed transaction: 0e49a421f6e75f4c5e09dd738a02d3f51bd18a0cf31894f68d335cd70d9c0e12  120 bytes  1000 cycles
	#   eosio.token <= eosio.token::create          {"issuer":"eosio","maximum_supply":"1000000000.0000 SYS"}

这个命令创建一个新的 token SYS, 有四位小数点精度及最大 1000000000.0000 SYS 供应量。

为了创建这个 token, 我们需要 eosio.token 合约的权限因为他 “拥有”符号名字空间（比如“SYS”）。这个合约的为了版本可能允许其它方自动购买符号名字。
因为这个原因我们必须传递 -p eosio.token@active 以授权这个调用。

授予 tokens 给 “user”账户

现在已经创建了 token, 发行人可以授予新 token 给之前创建的账户 user。如果你没有创建 user 账户，参见之前的指令。

我们将使用位置参数调用方式。

	$ cleos push action eosio.token issue '[ "user", "100.0000 SYS", "memo" ]' \
        -p eosio@active
	executed transaction: 822a607a9196112831ecc2dc14ffb1722634f1749f3ac18b73ffacd41160b019  268 bytes  1000 cycles
	#   eosio.token <= eosio.token::issue           {"to":"user","quantity":"100.0000 SYS","memo":"memo"}
	>> issue
	#   eosio.token <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}
	>> transfer
	#         eosio <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}
	#          user <= eosio.token::transfer        {"from":"eosio","to":"user","quantity":"100.0000 SYS","memo":"memo"}

这次输出包含几个不同的 actions: 一个 issue 及三个 transfers。虽然我们签署的唯一 action 是 issue, issue action 执行一次“在线转账”并且
“在线转账”通知发送方账户和接收方账户。输出指示了调用的所有 action 处理器，调用的顺序，action 是否有其它输出。

技术上， eosio.token 合约本可以忽略在线转账而直接修改余额。然而，这个例子，eosio.token 合约遵循 token 惯例，要求所有账户余额可以可以从 
transfer  action 总和中推导。也要求资金的发送方和接收方被通知以便他们能自动化处理存款和取款。

如果你想看广播的实际的 transaction, 可以用 -d -j 选项指示 “不要广播” 及 “以 json 格式返回 transaction”。

	$ cleos push action eosio.token issue '["user", "100.0000 SYS", "memo"]' -p eosio@active -d -j
	{
	  "expiration": "2018-05-25T19:02:58",
	  "ref_block_num": 18200,
	  "ref_block_prefix": 614206268,
	  "max_net_usage_words": 0,
	  "max_cpu_usage_ms": 0,
	  "delay_sec": 0,
	  "context_free_actions": [],
	  "actions": [{
	      "account": "eosio.token",
	      "name": "issue",
	      "authorization": [{
	          "actor": "eosio",
	          "permission": "active"
	        }
	      ],
	      "data": "00000000007015d640420f00000000000453595300000000046d656d6f"
	    }
	  ],
	  "transaction_extensions": [],
	  "signatures": [
	    "SIG_K1_Khyk1GsxWCx4axqYMF2AREDvaZtZdFaQifNPkR9DomR7toJ4sGua7pMBNq2osV5TY8rcGNcgNwn1eFe3noAXsoUA26HNDJ"
	  ],
	  "context_free_data": []
	}

转账 Tokens 给账户 “Tester”	
	
现在账户 user 已经有 tokens, 我们将转一些给账户 tester。我们指示 user 授权这次 action 用权限参数 -p user@active 。

	$ cleos push action eosio.token transfer \
	        '[ "user", "tester", "25.0000 SYS", "m" ]' -p user@active
	executed transaction: 06d0a99652c11637230d08a207520bf38066b8817ef7cafaab2f0344aafd7018  268 bytes  1000 cycles
	#   eosio.token <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}
	>> transfer
	#          user <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}
	#        tester <= eosio.token::transfer        {"from":"user","to":"tester","quantity":"25.0000 SYS","memo":"m"}


部署 Exchange 合约

类似于上面的例子，我们能部署 exchange 合约。exchange 合约提供了创建及交易货币的能力。假设从 EOSFORCEIO 源码的根目录运行。

	$ cleos create account eosio exchange  \
	        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 \
	        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
	executed transaction: 4d38de16631a2dc698f1d433f7eb30982d855219e7c7314a888efbbba04e571c  364 bytes  1000 cycles
	#         eosio <= eosio::newaccount            {"creator":"eosio","name":"exchange","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJxe...
	
	$ cleos set contract exchange build/contracts/exchange -p exchange@active
	Reading WAST/WASM from build/contracts/exchange/exchange.wasm...
	Using already assembled WASM...
	Publishing contract...
	executed transaction: 503dddec456ae301ef467c6a05bc6bf61e1ea21ab911ef6cc6e0750001b675c8  33888 bytes  5841 us
	#         eosio <= eosio::setcode               {"account":"exchange","vmtype":0,"vmversion":0,"code":"0061736d0100000001bb022f60067f7e7f7f7f7f00600...
	#         eosio <= eosio::setabi                {"account":"exchange","abi":"0e656f73696f3a3a6162692f312e30010c6163636f756e745f6e616d65046e616d650e0...

	
	
部署 Eosio.msig 合约

eosio.msig 合约允许多方异步签署单个 transaction。EOSFORCEIO 在基础层面提供多重签名支持，但是它需要一个同步的旁通道可以搬运数据及签署数据。
Eosio.msig 是一种更加友好的方式，可以异步建议，批准，及最终出版多方的同意。

采用下述命令部署 Eosio.msig 合约。

	$ cleos create account eosio eosio.msig  \
	        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 \
	        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4
	#         eosio <= eosio::newaccount            {"creator":"eosio","name":"eosio.msig","owner":{"threshold":1,"keys":[{"key":"EOS7ijWCBmoXBi3CgtK7DJ...
	
	$ cleos set contract eosio.msig build/contracts/eosio.msig -p eosio.msig@active
	Reading WAST/WASM from build/contracts/eosio.msig/eosio.msig.wasm...
	Using already assembled WASM...
	Publishing contract...
	executed transaction: 3433c434bdef42ba2150d5df46c17e0258f20b7836a057911faa2daa66262338  8864 bytes  1319 us
	#         eosio <= eosio::setcode               {"account":"eosio.msig","vmtype":0,"vmversion":0,"code":"0061736d010000000198011760017f0060047f7e7e7...
	#         eosio <= eosio::setabi                {"account":"eosio.msig","abi":"0e656f73696f3a3a6162692f312e30030c6163636f756e745f6e616d65046e616d650...

	
	
	
	

# “Hello World” 合约





# 多索引表使用指导




# 多索引表示例




# 在 EOSFORCEIO 上创建一个 Token




#  合约中随机化



# 升级系统合约




# 怎样写 ABI 文件


