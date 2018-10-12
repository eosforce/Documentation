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

然后我们可以部署合约，合约在 ${EOSIO_SOURCE}/build/contracts/eosio.token 目录下。

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



# 多索引表示例

描述

在这篇教程中我们将浏览在智能合约中创建和使用多索引表的步骤。

注释

多索引表是一种在内存中缓存状态和数据为了快速访问的方法。多索引表支持创建，读，更新和删除（CRUD）操作，有些区块链不支持（区块链仅支持创建和读）。

多索引表提供了一种快速访问的数据存储，是一种实用的在智能合约中存储数据的方式。区块链记录 transactions, 但你应该用多索引表存储应用数据。

它们是多索引表，因为它们支持在数据上使用多个索引。主键索引类型必须是 uint64_t 并且必须唯一，但是其它二级索引可以有重复。最多可以建16个索引并且
字段类型可以是 uint64_t, uint128_t, uint256_t, double 和 long double 。

如果你想在字符串上建索引，你需要将其转换成整数类型，并且将结果存储在一个字段上然后在其上建索引。


1、创建一个 struct

创建一个 struct , 它将被存在多索引表中，并且在想建索引的字段上定义 getters 。

记住这些 getters 其中之一必须名为 "primary_key()", 如果没有，编译器 （eosio-cpp）将生成一个错误。它无法发现字段用作主键。

如果你想建多个索引（最多允许16个），然后为想建索引的字段定义一个 getter, 这次名字不那么重要，因为你将传递 getter 名字进入 typedef 。

C++
	 struct [[eosio::table]] mystruct 
      {
         uint64_t     key; 
         uint64_t     secondid;
         std::string  name; 
         std::string  account; 

         uint64_t primary_key() const { return key; } // getter for primary key
         uint64_t by_id() const {return secondid; } // getter for additional key
      };

这儿要注意两件事：
1.属性 [[eosio::table]] 对 ABI 生成器是需要的，eosio-cpp, 识别出你想通过 ABI 暴露这个表并且使它在智能合约外可见。
2.struct 名称少于12个字符并且都是小写。


2. typedef 多索引表并且定义索引

定义将使用 mystruct 的多索引表，告诉它索引什么以及怎样获取被索引的数据。主键将被自动创建，因此使用上述 struct, 如果我要一个只有主键的多索引表
将定义如下：

C++

typedef eosio::multi_index<N(mystruct), mystruct> datastore;

这定义了多索引出入表名 N(mystruct) 和 struct 名 "mystruct"。N(mystruct) 执行一次编译转换从 struct 名到 uint64_t 并且这个 uint64_t
被用来识别属于这个多索引表的数据。

要增加二级索引，使用  indexed_by 模板做参数，因此定义变为：

C++

typedef eosio::multi_index<N(mystruct), mystruct, indexed_by<N(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>> datastore;

这儿

indexed_by<N(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>

参数

- 字段名转换为整数，N(secondid)
- 用户自定义键提取器， const_mem_fun<mystruct, uint64_t, &mystruct::by_id> 

如果想建3个索引

C++

	  struct [[eosio::table]] mystruct 
      {
         uint64_t     key; 
         uint64_t     secondid;
         uint64_t			anotherid;
         std::string  name; 
         std::string  account; 

         uint64_t primary_key() const { return key; }
         uint64_t by_id() const {return secondid; }
         uint64_t by_anotherid() const {return anotherid; }
      };
      
typedef eosio::multi_index<N(mystruct), mystruct, indexed_by<N(secondid), const_mem_fun<mystruct, uint64_t, &mystruct::by_id>>, indexed_by<N(anotherid), const_mem_fun<mystruct, uint64_t, &mystruct::by_anotherid>>> datastore;
      
      
诸此类推。

这儿有一件重要的事要注意，struct 名称匹配表名，并且名称将出现在 abi 文件，必须满足规则（少于12个字符并且都是小写）。如果不满足规则，这些表通过 abi 将
不可见(你可以通过编辑 abi 文件规避这个限制)。


3.创建定义类型的局部变量

	// local instances of the multi indexes
  	pollstable _polls;
    votes _votes;

现在我已经定义了一个有两个索引的多索引表，我可以在智能合约中用这个表。

下面展示一个用了两个多索引表的工作智能合约的例子。这儿你能看出怎样迭代表以及怎样在同一合约中用两个表。

C++

#include <eosiolib/eosio.hpp>

using namespace eosio;

class youvote : public contract {
  public:
      youvote(account_name s):contract(s), _polls(s, s), _votes(s, s)
      {}

      // public methods exposed via the ABI
      // on pollsTable

      [[eosio::action]]
      void version()
      {
          print("YouVote version  0.01"); 

      };
      
      [[eosio::action]]
      void addpoll(account_name s, std::string pollName)
      {
          //require_auth(s);

          print("Add poll ", pollName); 
              
          // update the table to include a new poll
          _polls.emplace(get_self(), [&](auto& p)
                                      {
                                        p.key = _polls.available_primary_key();
                                        p.pollId = _polls.available_primary_key();
                                        p.pollName = pollName;
                                        p.pollStatus = 0;
                                        p.option = "";
                                        p.count = 0;
                                      });
      };


      [[eosio::action]]
      void rmpoll(account_name s, std::string pollName)
      {
          //require_auth(s);

          print("Remove poll ", pollName); 
              
          std::vector<uint64_t> keysForDeletion;
          // find items which are for the named poll
          for(auto& item : _polls)
          {
              if (item.pollName == pollName)
              {
                  keysForDeletion.push_back(item.key);   
              }
          }
          
          // now delete each item for that poll
          for (uint64_t key : keysForDeletion)
          {
              print("remove from _polls ", key);
              auto itr = _polls.find(key);
              if (itr != _polls.end())
              {
                _polls.erase(itr);
              }
          }


          // add remove votes ... don't need it the actions are permanently stored on the block chain

          std::vector<uint64_t> keysForDeletionFromVotes;
          // find items which are for the named poll
          for(auto& item : _votes)
          {
              if (item.pollName == pollName)
              {
                  keysForDeletionFromVotes.push_back(item.key);   
              }
          }
          
          // now delete each item for that poll
          for (uint64_t key : keysForDeletionFromVotes)
          {
              print("remove from _votes ", key);
              auto itr = _votes.find(key);
              if (itr != _votes.end())
              {
                _votes.erase(itr);
              }
          }


      };

      [[eosio::action]]
      void status(std::string pollName)
      {
          print("Change poll status ", pollName);

          std::vector<uint64_t> keysForModify;
          // find items which are for the named poll
          for(auto& item : _polls)
          {
              if (item.pollName == pollName)
              {
                  keysForModify.push_back(item.key);   
              }
          }
          
          // now get each item and modify the status
          for (uint64_t key : keysForModify)
          {

            print("modify _polls status", key);
            auto itr = _polls.find(key);
            if (itr != _polls.end())
            {
              _polls.modify(itr, get_self(), [&](auto& p)
                                              {
                                                p.pollStatus = p.pollStatus + 1;
                                              });
            }
          }
      };

      [[eosio::action]]
      void statusreset(std::string pollName)
      {
          print("Reset poll status ", pollName); 
              
          std::vector<uint64_t> keysForModify;
          // find all poll items
          for(auto& item : _polls)
          {
              if (item.pollName == pollName)
              {
                  keysForModify.push_back(item.key);   
              }
          }
          
          // update the status in each poll item
          for (uint64_t key : keysForModify)
          {
              print("modify _polls status", key);
              auto itr = _polls.find(key);
              if (itr != _polls.end())
              {
                _polls.modify(itr, get_self(), [&](auto& p)
                                                {
                                                  p.pollStatus = 0;
                                                });
              }
          }
      };


      [[eosio::action]]
      void addpollopt(std::string pollName, std::string option)
      {
          print("Add poll option ", pollName, "option ", option); 

          // find the pollId, from _polls, use this to update the _polls with a new option
          for(auto& item : _polls)
          {
              if (item.pollName == pollName)
              {
                    // can only add if the poll is not started or finished
                    if(item.pollStatus == 0)
                    {
                        _polls.emplace(get_self(), [&](auto& p)
                                          {
                                            p.key = _polls.available_primary_key();
                                            p.pollId = item.pollId;
                                            p.pollName = item.pollName;
                                            p.pollStatus = 0;
                                            p.option = option;
                                            p.count = 0;
                                          });
                    }
                    else
                    {
                        print("Can not add poll option ", pollName, "option ", option, " Poll has started or is finished.");
                    }

                    break; // so you only add it once
              }
          }
      };

      [[eosio::action]]
      void rmpollopt(std::string pollName, std::string option)
      {
          print("Remove poll option ", pollName, "option ", option); 
              
          std::vector<uint64_t> keysForDeletion;
          // find and remove the named poll
          for(auto& item : _polls)
          {
              if (item.pollName == pollName)
              {
                  keysForDeletion.push_back(item.key);   
              }
          }
          
          
          for (uint64_t key : keysForDeletion)
          {
              print("remove from _polls ", key);
              auto itr = _polls.find(key);
              if (itr != _polls.end())
              {
                  if (itr->option == option)
                  {
                      _polls.erase(itr);
                  }
              }
          }
      };


      [[eosio::abi]]
      void vote(std::string pollName, std::string option, std::string accountName)
      {
          print("vote for ", option, " in poll ", pollName, " by ", accountName); 

          // is the poll open
          for(auto& item : _polls)
          {
              if (item.pollName == pollName)
              {
                  if (item.pollStatus != 1)
                  {
                      print("Poll ",pollName,  " is not open");
                      return;
                  }

                  break; // only need to check status once
              }
          }

          // has account name already voted?  
          for(auto& vote : _votes)
          {
              if (vote.pollName == pollName && vote.account == accountName)
              {
                  print(accountName, " has already voted in poll ", pollName);
                  //eosio_assert(true, "Already Voted");
                  return;
              }
          }

          uint64_t pollId =99999; // get the pollId for the _votes table

          // find the poll and the option and increment the count
          for(auto& item : _polls)
          {
              if (item.pollName == pollName && item.option == option)
              {
                  pollId = item.pollId; // for recording vote in this poll

                  _polls.modify(item, get_self(), [&](auto& p)
                                                {
                                                    p.count = p.count + 1;
                                                });
              }
          }

          // record that accountName has voted
          _votes.emplace(get_self(), [&](auto& pv)
                                      {
                                        pv.key = _votes.available_primary_key();
                                        pv.pollId = pollId;
                                        pv.pollName = pollName;
                                        pv.account = accountName;
                                      });        
      };

  private:    

    // create the multi index tables to store the data
      struct [[eosio::table]] poll 
      {
        uint64_t      key; // primary key
        uint64_t      pollId; // second key, non-unique, this table will have dup rows for each poll because of option
        std::string   pollName; // name of poll
        uint8_t      pollStatus =0; // staus where 0 = closed, 1 = open, 2 = finished
        std::string  option; // the item you can vote for
        uint32_t    count =0; // the number of votes for each itme -- this to be pulled out to separte table.

        uint64_t primary_key() const { return key; }
        uint64_t by_pollId() const {return pollId; }
      };
      typedef eosio::multi_index<N(poll), poll, indexed_by<N(pollId), const_mem_fun<poll, uint64_t, &poll::by_pollId>>> pollstable;

      struct [[eosio::table]] pollvotes 
      {
         uint64_t     key; 
         uint64_t     pollId;
         std::string  pollName; // name of poll
         std::string  account; //this account has voted, use this to make sure noone votes > 1

         uint64_t primary_key() const { return key; }
         uint64_t by_pollId() const {return pollId; }
      };
      typedef eosio::multi_index<N(pollvotes), pollvotes, indexed_by<N(pollId), const_mem_fun<pollvotes, uint64_t, &pollvotes::by_pollId>>> votes;

      // local instances of the multi indexes
      pollstable _polls;
      votes _votes;
};

EOSIO_ABI( youvote, (version)(addpoll)(rmpoll)(status)(statusreset)(addpollopt)(rmpollopt)(vote))


注意 EOSIO_ABI 调用，这通过 abi 暴露函数，函数名必须匹配 abi 函数名。



# 在 EOSFORCEIO 上创建一个 Token

在这个阶段区块链并不能做很多，因此让我们部署 eosio.token 合约。这个合约使能创建许多不同的 tokens, 都运行在相同的合约上但是潜在被不同的用户管理。

在我们部署 token 合约前，先要将要部署上去的账户。

	$ cleos create account eosio eosio.token \
        EOS7ijWCBmoXBi3CgtK7DJxentZZeTkeUnaSDvyro9dq7Sd1C3dC4 \
	...

然后我们可以部署合约，合约在 ${EOSIO_SOURCE}/build/contracts/eosio.token 目录下。

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


#  合约中随机化

通过使用 openssl, sha256 及 checksum256，一个确定及有效随机的值能被生成。

第1步： 生成 Secrets

Shell

openssl rand -hex 32 
28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905

openssl rand -hex 32
15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12


第2步： 生成 sha256 哈希

Shell

echo -n '28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905' | xxd -r -p | sha256sum -b | awk '{print $1}'
d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883

echo -n '15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12' | xxd -r -p | sha256sum -b | awk '{print $1}'
50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129

第3步： 提交 Secret 的哈希

Alice 和 Bob 每人提交他们的 Secret 的哈希。

Shell

cleos push action chance submithash '[ "alice", "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883" ]' -p alice@active	

cleos push action chance submithash '[ "bob", "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129" ]' -p bob@active

第4步： 提交 Secret

lice 和 Bob 每人提交他们的 Secret 的哈希及 Secret。

Shell

cleos push action chance submitboth '[ "d533f24d6f28ddcef3f066474f7b8355383e485681ba8e793e037f5cf36e4883", "28349b1d4bcdc9905e4ef9719019e55743c84efa0c5e9a0b077f0b54fcd84905" ]' -p alice@active

cleos push action chance submitboth '[ "50ed53fcdaf27f88d51ea4e835b1055efe779bb87e6cfdff47d28c88ffb27129", "15fe76d25e124b08feb835f12e00a879bd15666a33786e64b655891fba7d6c12" ]' -p bob@active


第5步： 结论

能存储 secret 及 secret 哈希的最简单的数据类型是一个 struct 。

C++

struct player {
  checksum256 hash;
  checksum256 secret;
};

当游戏的玩家实例化时，重要的是实例化要连续。这样所有玩家的 secrets 及哈希连续存储在内存中。

C++

struct game {
  uint64_t id;
  player player1;
  player pla
}

当收到所有玩家的 secret 及 secret 的哈希时，能完成整体的哈希。

C++

// variable to get result from hashing all players hashes and secrets
checksum256 result;

// hash the contents in memory, starting at game.player1 and spanning for 
// sizeof(player)*2 bytes
sha256( (char *)&game.player1, sizeof(player)*2, &result);

// compares first and second 4 byte chunks in result to determine a winner
int winner = result.hash[1] < result.hash[0] ? 0 : 1;

// report appropriate winner
if( winner ) {
  report_winner(game.player1);
} else {
  report_winner(game.player2);
}

注意 sha256 是32字节，有 8个 uint32_t 组成。上面的例子中，考虑了前两个元素，但是可以是任意一个。


# 升级系统合约

使用 eosio.msig 合约的间接方法




直接方法（避免使用 eosio.msig 合约）

前21个区块生产者中每一个应该做下面的事：

1.获取当前系统合约为了以后的比较（主网区块链上实际的哈希及 ABI将不同）

	$ programs/cleos/cleos get code -c original_system_contract.wast -a original_system_contract.abi eosio
	code hash: cc0ffc30150a07c487d8247a484ce1caf9c95779521d8c230040c2cb0e2a3a60
	saving wast to original_system_contract.wast
	saving abi to original_system_contract.abi

2.生成升级系统合约的未签署 transaction

	$ programs/cleos/cleos set contract -s -j -d eosio contracts/eosio.system | tail -n +4 > upgrade_system_contract_trx.json

生成的文件前几行类似于（除了 expiration, ref_block_num, 及 ref_block_prefix 差异比较大）：

	{
   		"expiration": "2018-06-15T22:17:10",
   		"ref_block_num": 4552,
   		"ref_block_prefix": 511016679,
   		"max_net_usage_words": 0,
   		"max_cpu_usage_ms": 0,
   		"delay_sec": 0,
   		"context_free_actions": [],
   		"actions": [{
   		   "account": "eosio",
   		   "name": "setcode",
   		   "authorization": [{
   		       "actor": "eosio",
   		       "permission": "active"
   		     }
   		   ],

最后几行应该是：

		      }
	   ],
	   "transaction_extensions": [],
	   "signatures": [],
	   "context_free_data": []
	}

顶级区块生产者之一应该被选出来领导整个升级过程。这个领导生产者应该生成它的 upgrade_system_contract_trx.json，改名为
upgrade_system_contract_official_trx.json，然后执行下面动作：

3.修改文件中 expiration 时间戳为一个足够远的时间，能完成收集所有必需的签名, 但是距生成 transaction 时刻不超过9小时。
并且记住 transaction 将不被接纳进入区块链，如果 expiration 距当前时间超过1小时。

4.传递 upgrade_system_contract_official_trx.json 文件给其他21个顶级区块生产者。

然后21个顶级区块生产者中每一个应该执行下面的动作：

5.比较生成的 upgrade_system_contract_official_trx.json 文件和领导生产者提供的 upgrade_system_contract_official_trx.json 。
唯一的区别应该是 expiration, ref_block_num, ref_block_prefix， 例如：

	$ diff upgrade_system_contract_official_trx.json upgrade_system_contract_trx.json
	2,4c2,4
	<   "expiration": "2018-06-15T22:17:10",
	<   "ref_block_num": 4552,
	<   "ref_block_prefix": 511016679,
	---
	>   "expiration": "2018-06-15T21:20:39",
	>   "ref_block_num": 4972,
	>   "ref_block_prefix": 195390844,

6.如果比较OK，每个区块生产者应该使用满足 active 权限的密钥签署正式的升级 transaction。如果区块生产者账户中 active 权限中仅有单个私钥
（比如 “active key”）, 然后他们仅需要使用 active key 生成一次签名。这个签名过程可以离线完成为了额外安全性。

首先区块生产者应该收集所有必须信息。我们假设区块生产者 active 密钥对为
(EOS5kBmh5kfo6c6pwB8j77vrznoAaygzoYvBsgLyMMmQ9B6j83i9c, 5JjpkhxAmEfynDgSn7gmEKEVcBqJTtu6HiQFf4AVgGv5A89LfG3)。区块生产者需要 active 私钥
（这个例子中是 5JjpkhxAmEfynDgSn7gmEKEVcBqJTtu6HiQFf4AVgGv5A89LfG3）， upgrade_system_contract_official_trx.json， chain_id
(这个例子中是 d0242fb30b71b82df9966d10ff6d09e4f5eb6be7ba85fd78f796937f1959315e, 可以通过 cleos get info 命令获取)。

然后在一台安全的计算机上生产者可以签署 transaction (生产者被提示时需要输入私钥)：

	$ programs/cleos/cleos sign --chain-id d0242fb30b71b82df9966d10ff6d09e4f5eb6be7ba85fd78f796937f1959315e upgrade_system_contract_trx.json | tail -n 5
	private key:   "signatures": [
	    "SIG_K1_JzABB9gzDGwUHaRmox68UNcfxMVwMnEXqqS1MvtsyUX8KGTbsZ5aZQZjHD5vREQa5BkZ7ft8CceLBLAj8eZ5erZb9cHuy5"
	  ],
	  "context_free_data": []
	}

确保使用 transaction 将要被提交到的 主网区块链 chain_id, 而不是上面例子中的 chain_id。

输出中包含签名（这个例子中是 SIG_K1_JzABB9gzDGwUHaRmox68UNcfxMVwMnEXqqS1MvtsyUX8KGTbsZ5aZQZjHD5vREQa5BkZ7ft8CceLBLAj8eZ5erZb9cHuy5），
生产者将发送给领导生产者。

当领导生产者收集了15个生产者签名时，领导生产者应该执行下列动作：

7.拷贝 upgrade_system_contract_official_trx.json 并命名为 upgrade_system_contract_official_trx_signed.json，然后修改
upgrade_system_contract_official_trx_signed.json 以便 signatures 字段包含15个收集到的签名。因此 
upgrade_system_contract_official_trx_signed.json 文件的尾部可能看起来像：

	$ cat upgrade_system_contract_official_trx_signed.json | tail -n 20
	  "transaction_extensions": [],
	  "signatures": [
	   "SIG_K1_JzABB9gzDGwUHaRmox68UNcfxMVwMnEXqqS1MvtsyUX8KGTbsZ5aZQZjHD5vREQa5BkZ7ft8CceLBLAj8eZ5erZb9cHuy5",
	   "SIG_K1_Kj7XJxnPQSxEXZhMA8uK3Q1zAxp7AExzsRd7Xaa7ywcE4iUrhbVA3B6GWre5Ctgikb4q4CeU6Bvv5qmh9uJjqKEbbjd3sX",
	   "SIG_K1_KbE7qyz3A9LoQPYWzo4e6kg5ZVojQVAkDKuufUN2EwVUqtFhtjmGoC6QPQqLi8J7ftiysBp52wJBPjtNQUfZiGpGMsnZ1f",
	   "SIG_K1_KdQsE7ahHA9swE9SDGg4oF6XahpgHmZfEgQAy9KPBLd9HuwrF6c8m6jz43zizK2oo32Ejg1DYuMfoEvJgVfXo81jsqTHvA",
	   "SIG_K1_K6228Hi2z1WabgVdf5bk2UdKyyDSVFwkMaagTn9oLVDV8rCX7aQcjY94c39ah2CkLTsTEqzTPAYknJ8m2m9B7npPkHaFzc",
	   "SIG_K1_Jzdx75hBCA2WSaXgrupmrNbcQocUCsP8r1BKkPXMreiAKPZDwX9J3G8fS1HhyqWjc7FbukwZf8sVRdS3wKbJVpytqXe7Nn",
	   "SIG_K1_KW7Qu2SdPD3zuQKh2ziFLzn9QbKqeMpeiemULky5Bbg1Mst6ijbCX3k2AVFGNFLkNLA36PM1WAT5oipzu1B1K7ymRxTx1Z",
	   "SIG_K1_KXJf1KZNpz73YFKKE7u6jFgsQ8XcX3yA7rDX6ZmG1Qfnc9FLLmT1WViv4bwcPbxaEYfR6SNWfk5cCR9eao2si1soqkXq92",
	   "SIG_K1_JynjkHFT5UFGDpEcqdriXTzCGCwS36Xztq4UAWQHLQgRUZT2YFoLhUcc87kvUteqCUGVxsmSbfgWv1KLy24voKN4Qs5zTe",
	   "SIG_K1_JxhfCaGBhuNShpDHn7j1CryG3iSebvfi7FUnJsfkXNTiwLyq2NDBkeakwjCMWFbzr6qqWuMDLjfXbzdtU17f1wCXMjKSgk",
	   "SIG_K1_KcMSz89QG1ZRFNrXc69R63d5KXbJA8CBjNPYv1VEA3TRfjqVYuhyaHpGXQN4RSKDq4ygr3UTRYBQQVutkJnR6zZ4Ssgd7R",
	   "SIG_K1_JuxT6bhUAbDs6Q2ppuKyKauduvbaJLxvh4gBH4e4A9yRhvUBT7w3DcvMyhdaor27Kbu29jnqhTbvXcb57QqKWQDpboLv7e",
	   "SIG_K1_K8BuFYpCiC5FhpVK8ZAzc3VUg7vz6WwLoWBrGN6nnuqUjngGqvHp3UxDVzcwhqccHdv8kdPXvF6G1NszwF1dd3wjCrHBYw",
	   "SIG_K1_KfH5ZirPwDk1RQKvJv2AGPfsJyPXvXLegZ7LvcPmRtjtMiErs1STXLNT8kiBfhZr4xkWRA5NR1kMF3d49DFMJiB2iWMXJc",
	   "SIG_K1_KjJB8jtcqpVe3r5jouFiAa9wJeYqoLMh5xrUV6kBF6UWfbYjimMWBJWz2ZPomGDsk7JtdUESVrYj1AhYbdp3X48KLm5Cev"
	  ],
	  "context_free_data": []
	}

8.推送签署的 transaction 进入区块链

	$ programs/cleos/cleos push transaction --skip-sign upgrade_system_contract_official_trx_signed.json
	{
	  "transaction_id": "202888b32e7a0f9de1b8483befac8118188c786380f6e62ced445f93fb2b1041",
	  "processed": {
	    "id": "202888b32e7a0f9de1b8483befac8118188c786380f6e62ced445f93fb2b1041",
	    "receipt": {
	      "status": "executed",
	      "cpu_usage_us": 4909,
	      "net_usage_words": 15124
	    },
	    "elapsed": 4909,
	    "net_usage": 120992,
	    "scheduled": false,
	    "action_traces": [{
	...

如果得到一个类似下面的错误：

	Error 3090003: provided keys, permissions, and delays do not satisfy declared authorizations
	Ensure that you have the related private keys inside your wallet and your wallet is unlocked.

意味着至少有一个签名是错的。这可能是因为某个生产者签署了错误的 transaction, 使用了错误的私钥，或者使用了错误的 chain_id 。

如果得到一个类似下面的错误消息：

	Error 3090002: irrelevant signature included
	Please remove the unnecessary signature from your transaction!

这意味着包含了不必要的签名。如果有21个活跃生产者，仅需要15个活跃生产者签名。

如果得到一个类似下面的错误消息：

	Error 3040006: Transaction Expiration Too Far
	Please decrease the expiration time of your transaction!

这意味着 expiration 时间超过1小时并且在推送 transaction 前你需要等待一些时间。

如果得到一个类似下面的错误消息：

	Error 3040005: Expired Transaction
	Please increase the expiration time of your transaction!

这意味着已经超过了签署 transaction 的超时时间并且整个过程必须从第1步重新开始。

9.假设 transaction 成功执行，每个人都能验证合约是否合适：

	$ programs/cleos/cleos get code -c new_system_contract.wast -a new_system_contract.abi eosio
	code hash: 9fd195bc5a26d3cd82ae76b70bb71d8ce83dcfeb0e5e27e4e740998fdb7b98f8
	saving wast to new_system_contract.wast
	saving abi to new_system_contract.abi
	$ diff original_system_contract.abi new_system_contract.abi
	584,592d583
	<         },{
	<           "name": "deferred_trx_id",
	<           "type": "uint32"
	<         },{
	<           "name": "last_unstake_time",
	<           "type": "time_point_sec"
	<         },{
	<           "name": "unstaking",
	<           "type": "asset"


# 怎样写 ABI 文件

介绍

前面我们用提供的 ABI 文件部署了 eosio.token 合约。这个教程概要描述了 ABI 文件怎样跟 eosio.token 合约关联。

ABI 文件可以用 eosio.cdt 提供的 eosiocpp 工具生成。然而有几种场景会导致 ABI 生成工作不正常或彻底失败。
高级 C++ 模式能导致生成失败，自定义类型有时业务导致生成 ABI 出问题。因为这个原因，你必须理解 ABI 文件怎么工作的，以便需要时你能调试和修复。

什么是 ABI？

Application Binary Interface (ABI) 是基于 JSON 的描述，描述怎样在 JSON 和 二进制表示之间转换用户 actions。ABI 也描述了怎样将数据库状态转换
到/从 JSON。一旦你通过 ABI 描述了合约，然后开发者和用户就能通过 JSON 与合约进行无缝交互。

安全提示

执行 transaction 时 ABI 可以被忽略。传给合约的消息和 actions 不一定遵循 ABI。ABI 是指导，而不是守门人。

创建 ABI 文件

从一个空 ABI 开始，命名为 eosio.token.abi

{
   "version": "eosio::abi/1.0",
   "types": [],
   "structs": [],
   "actions": [],
   "tables": [],
   "ricardian_clauses": [],
   "abi_extensions": [],
   "___comment" : ""
}

类型

ABI 使任何客户端或接口能为合约翻译及生成 GUI。为了这个能以一致方式工作，描述将在公共 action 或 struct 中用作参数的自定义类型。

内建类型

EOSIO 实现了很多内建类型。内建类型不需要在 ABI 文件中描述。如果你想熟悉 EOSIO 内建类型，它们定义在这儿。

以 eosio.token 为例，需要在 ABI 文件中描述的唯一的类型是 account_name。ABI 使用 "new_type_name" 描述显式类型，这个例子中是
account_name，account_name 是 name 类型的别名。

下面是描述这个类型的 JSON：

JSON

{
   "new_type_name": "account_name",
   "type": "name"
}

ABI 现在看起来像这样：

{
   "version": "eosio::abi/1.0",
   "types": [{
     "new_type_name": "account_name",
     "type": "name"
	 }],
   "structs": [],
   "actions": [],
   "tables": [],
   "ricardian_clauses": [],
   "abi_extensions": []
}

Structs

暴露到 ABI 中的 Structs 也需要被描述。通过查看 eosio.token.hpp， 我能快速判定哪些 structs 被公共 actions 使用。这对下一步特别重要。

JSON 中 struct 对象定义看起来像下面：

JSON

{
     "name": "issue", //The name 
     "base": "", 			//Inheritance, parent struct
     "fields": []			//Array of field objects describing the struct's fields. 
}

Fields

JSON

{
	"name":"", 	// The field's name
  	"type":""   // The field's type
}   

eosio.token 合约有许多 structs 需要定义。请注意，并非所有 structs 被显式定义，有些对应 actions 的参数。这儿是一份  eosio.token 合约
需要 ABI 描述的 structs 列表：


隐式 Structs

下面的 structs 是隐式的，因为struct 从未显式地在合约中定义。查看 create action, 你将发现两个参数，类型为 account_name 的 issuer
及类型为 asset 的 maximum_supply。为了简洁，本教程不打算分解每个 struct, 而是应用相同的逻辑，类似下面这样：

create

JSON

{
  "name": "create",
  "base": "",
  "fields": [
    {
      "name":"issuer", 
      "type":"account_name"
    },
    {
      "name":"maximum_supply", 
      "type":"asset"
    }
  ]
}

issue

JSON

{
  "name": "issue",
  "base": "",
  "fields": [
    {
      "name":"to", 
      "type":"account_name"
    },
    {
      "name":"quantity", 
      "type":"asset"
    },
    {
      "name":"memo", 
      "type":"string"
    }
  ]
}

retire

JSON

{
  "name": "retire",
  "base": "",
  "fields": [
    {
      "name":"quantity", 
      "type":"asset"
    },
    {
      "name":"memo", 
      "type":"string"
    }
  ]
}

transfer

JSON

{
  "name": "transfer",
  "base": "",
  "fields": [
    {
      "name":"from", 
      "type":"account_name"
    },
    {
      "name":"to", 
      "type":"account_name"
    },
    {
      "name":"quantity", 
      "type":"asset"
    },
    {
      "name":"memo", 
      "type":"string"
    }
  ]
}

close

JSON
{
  "name": "close",
  "base": "",
  "fields": [
    {
      "name":"owner", 
      "type":"account_name"
    },
    {
      "name":"symbol", 
      "type":"symbol"
    }
  ]
}


显式 Structs

这些 structs 显式定义，因为它们是实例化多索引表的必要条件。描述它们跟上面定义的隐式 structs 没什么不同。

account

JSON

{
  "name": "account",
  "base": "",
  "fields": [
    {
      "name":"balance", 
      "type":"asset"
    }
  ]
}

currency_stats

JSON

{
  "name": "currency_stats",
  "base": "",
  "fields": [
    {
      "name":"supply", 
      "type":"asset"
    },
    {
      "name":"max_supply", 
      "type":"asset"
    },
    {
      "name":"issuer", 
      "type":"account_name"
    }
  ]
}



Actions

action 的JSON 对象定义看起来像下面：

JSON

{
  "name": "transfer", 			//The name of the action as defined in the contract
  "type": "transfer", 			//The name of the implicit struct as described in the ABI
  "ricardian_contract": "" 	//An optional ricardian clause to associate to this action describing its intended functionality.
}


通过聚集 eosio.token 合约头文件中所有公共函数描述 eosio.token 合约的 actions。

然后按照之前描述的 struct 描述每个 action 的类型。绝大多数情况下，函数名跟 struct 名相同，但不要求相同。

下面是 actions 列表，链接到它们的源代码，并附带示例 JSON 用于阐明每个 action 怎样被描述。

create

JSON

{
  "name": "create",
  "type": "create",
  "ricardian_contract": ""
}

issue

JSON

{
  "name": "issue",
  "type": "issue",
  "ricardian_contract": ""
}

retire

JSON

{
  "name": "retire",
  "type": "retire",
  "ricardian_contract": ""
}

transfer

JSON

{
  "name": "transfer",
  "type": "transfer",
  "ricardian_contract": ""
}

close

JSON

{
  "name": "close",
  "type": "close",
  "ricardian_contract": ""
}


表

描述表。这是表的 JSON 对象定义：

JSON

{
  "name": "",       //The name of the table, determined during instantiation. 
  "type": "", 			//The table's corresponding struct
  "index_type": "", //The type of primary index of this table
  "key_names" : [], //An array of key names, length must equal length of key_types member
  "key_types" : []  //An array of key types that correspond to key names array member, length of array must equal length of key names array.
}

eosio.token 合约实例化两张表，accounts 和 stat。

accounts 表是 i64 索引，基于 account struct， 有一个 uint64 作为主键，并且键名任意命名为 “currency”。

下面是描述在 ABI 文件中的 accounts 表。

{
  "name": "accounts",
  "type": "account", // Corresponds to previously defined struct
  "index_type": "i64",
  "key_names" : ["currency"],
  "key_types" : ["uint64"]
}

stat 表是 i64 索引，基于 currenct_stats struct， 有一个 uint64 作为主键，并且键名任意命名为 “currency”。

下面是描述在 ABI 文件中的 stat 表。

{
  "name": "stat",
  "type": "currency_stats",
  "index_type": "i64",
  "key_names" : ["currency"],
  "key_types" : ["uint64"]
}

你会注意到上面的表有相同的“键名”。给 keys 命名类似地名称是象征性的，潜在暗示了一种主观联系。正如这个实现，意味zhe
任何不同的值可以用来查询不同的表。

综合起来

最后，一个准确描述 eosio.token 合约的 ABI 文件。

JSON

{
  "version": "eosio::abi/1.0",
  "types": [
    {
      "new_type_name": "account_name",
      "type": "name"
    }
  ],
  "structs": [
    {
      "name": "create",
      "base": "",
      "fields": [
        {
          "name":"issuer", 
          "type":"account_name"
        },
        {
          "name":"maximum_supply", 
          "type":"asset"
        }
      ]
    },
    {
       "name": "issue",
       "base": "",
       "fields": [
          {
            "name":"to", 
            "type":"account_name"
          },
          {
            "name":"quantity", 
            "type":"asset"
          },
          {
            "name":"memo", 
            "type":"string"
          }
       ]
    },
    {
       "name": "retire",
       "base": "",
       "fields": [
          {
            "name":"quantity", 
            "type":"asset"
          },
          {
            "name":"memo", 
            "type":"string"
          }
       ]
    },
    {
       "name": "close",
       "base": "",
       "fields": [
          {
            "name":"owner", 
            "type":"account_name"
          },
          {
            "name":"symbol", 
            "type":"symbol"
          }
       ]
    },
    {
      "name": "transfer",
      "base": "",
      "fields": [
        {
          "name":"from", 
          "type":"account_name"
        },
        {
          "name":"to", 
          "type":"account_name"
        },
        {
          "name":"quantity", 
          "type":"asset"
        },
        {
          "name":"memo", 
          "type":"string"
        }
      ]
    },
    {
      "name": "account",
      "base": "",
      "fields": [
        {
          "name":"balance", 
          "type":"asset"
        }
      ]
    },
    {
      "name": "currency_stats",
      "base": "",
      "fields": [
        {
          "name":"supply", 
          "type":"asset"
        },
        {
          "name":"max_supply", 
          "type":"asset"
        },
        {
          "name":"issuer", 
          "type":"account_name"
        }
      ]
    }
  ],
  "actions": [
    {
      "name": "transfer",
      "type": "transfer",
      "ricardian_contract": ""
    },
    {
      "name": "issue",
      "type": "issue",
      "ricardian_contract": ""
    },
    {
      "name": "retire",
      "type": "retire",
      "ricardian_contract": ""
    },
    {
      "name": "create",
      "type": "create",
      "ricardian_contract": ""
    },
    {
      "name": "close",
      "type": "close",
      "ricardian_contract": ""
    }
  ],
  "tables": [
    {
      "name": "accounts",
      "type": "account",
      "index_type": "i64",
      "key_names" : ["currency"],
      "key_types" : ["uint64"]
    },
    {
      "name": "stat",
      "type": "currency_stats",
      "index_type": "i64",
      "key_names" : ["currency"],
      "key_types" : ["uint64"]
    }
  ],
  "ricardian_clauses": [],
  "abi_extensions": []
}


Token 合约没有涉及的案例

Vectors

当在 ABI 文件中描述 vector 时，简单地在类型后附加 [] , 因此如果你需要描述权限级别 vector, 你将这样描述：permission_level[]

Struct Base

这是一个很少使用的属性。你可以用 base ABI struct 属性引用另一个 struct 为了继承，只要这个 struct 也描述在同样的 ABI 文件中。
Base 不做任何事或潜在抛出错误，如果你的智能合约逻辑不支持继承。

你可以在系统合约源代码和 ABI 中看到 base 的示例。



其它这儿没有涉及的 ABI 属性

为了简洁 ABI 规范的一些属性这儿省略了，然而，有一份还未完成的 ABI 规范将完整列出 ABI 每一个属性。

理查德条款

理查德条款描述了特定 actions 的目标输出。也可以用来建立发送者与合约之间的条款。


ABI 扩展

一个泛型的“未来证明”层允许老客户端忽略解析扩展数据块。现在，这个属性未使用。将来每个扩展将有自己的块以便老客户端忽略它，新客户端能理解怎样翻译它。


维护

每次你改变一个 struct, 增加一个表，增加一个 action 或者给 action 增加一个参数，使用一个新类型，你需要记得更新 ABI 文件。在许多情况下
不更新 ABI 文件并不会产生错误。

排查故障

表不返回行

检查你的表是否准确的在 ABI 文件中描述。例如，如果你用 cleos 在合约中增加了一个表用错误的格式，然后到表中查询数据，你将收到一个空结果。
如果合约没有合适地在 ABI 文件中描述表，当增加一行或读取一行时， cleos 不会生成错误。




