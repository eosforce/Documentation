# 介绍

现实世界的合约简而言之是一种给定一套输入规定行为产出的协议。一份合约可以是一份正式的法律合约（比如金融交易），也可以简单如某种游戏规则。
典型的actions诸如转账（在金融交易场景下）、游戏移动（在游戏合约场景下）。

EOS原力智能合约是一种软件，在区块链上注册，在EOS原力节点上执行，实现了合约的语义, action 请求账本存放在区块链上。智能合约定义了接口
（a(actions, parameters, data structures) 以及实现接口的代码。
代码编译成节点能获取和执行的标准字节码格式。区块链存储合约的transactions(例如转账、游戏移动)。每份智能合约伴随一份Ricardian Contract，其定义了
合约绑定的条款和条件。


# 需要的知识

C / C++ 经验
基于EOS原力的区块链执行用户生成的WebAssembly(WASM)应用和代码。WASM是一种新兴的web标准，得到了Google,Mirosoft,Apple等公司的广泛支持。
当前构建编译成WASM的应用最成熟的工具链是 clang/llvm 自带的 C/C++编译器。为了最好的兼容性，推荐使用EOS原力工具链。

其它第三方开发工具链包括：Rust, Python, and Solidity。虽然这些其它语言看起来更简单，但是它们的性能可能影响你构建应用的规模。我们期望C++成为
开发高性能及安全智能合约的最好的语言，并且计划在可预见的将来采用C++。

Linux / Mac 操作系统经验
EOS原力软件支持下列环境：

    Amazon 2017.09 and higher
    Centos 7
    Fedora 25 and higher (Fedora 27 recommended)
    Mint 18
    Ubuntu 16.04 (Ubuntu 16.10 recommended)
    Ubuntu 18.04 LTS
    MacOS Darwin 10.12 and higher (MacOS 10.13.x recommended)

命令行知识
EOS原力自带很多工具，为了能和他们交互需要你有基本的命令行知识。


# 通讯模型
EOS智能合约保护一套action和type定义。Action定义规范和实现合约的行为。type定义规范需要的内容和结构。EOS原力主要运行在基于消息的通讯框架下。
客户端通过发送（推送）消息给nodeos来触发actions。可以通过cleos命令来完成。也可以通过EOSFORCEIO send 方法（比如eosio::action::send）。
nodeos派发action 请求给实现合约的WASM 代码。合约代码作为一个整体运行，然后继续转到下个action处理。

EOSFORCEIO智能合约可以互相通信，比如让另一个合约在当前transaction完成后执行某些操作，或者在当前transaction作用域外触发一个未来执行的transaction。

EOSFORCEIO支持两种基本通信模型，inline 和 deferred。在当前transaction内执行的操作是 inline action 的示例，而触发未来transaction是deferred action的示例。

合约间通讯应当看作异步发生。异步通讯模型可能导致垃圾消息，这由资源限制算法来解决。

inline 通讯

inline 通讯呈现的形式为：请求其它actions作为调用action的一部分来执行。inline actions和原始的transaction运行在相同的作用域及授权下，并且保证和当前transaction一起执行。
这些可以有效的看作是调用transction内的嵌套transaction。如果transaction的任何部分失败，inline actions将和transaction的其它部分一起解约。调用inline action除了返回成功或失败
不会生成任何通知。

deferred 通讯

deferred 通讯概念上以发送通知给对端tranaction的形式呈现。deferred actions由生产者决定在以后的某个时间运行。不保证deferred action一定会被执行。

如前所述，deferred 通讯由生产者决定被调度到以后执行。从发起transaction（比如创建deferred transaction的 transaction）的角度来看，它只能判定
创建请求是否成功提交或没有成功提交（如果没有成功提交，会立即返回失败）。deferred transaction携带了合约的授权信息。transaction能取消deferred transaction。

Transactions VS. Actions

action表示单个操作，而transaction是一个或多个actions的组合。合约和账户以actions的形式通讯。actions可以单独发送，也可以合并发送，如果他们期望被当做整体执行。

只有一个action的transaction

{
  "expiration": "2018-04-01T15:20:44",
  "region": 0,
  "ref_block_num": 42580,
  "ref_block_prefix": 3987474256,
  "net_usage_words": 21,
  "kcpu_usage": 1000,
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
      "data": "00000000007015d640420f000000000004454f5300000000046d656d6f"
    }
  ],
  "signatures": [
    ""
  ],
  "context_free_data": []
}

有多个actions的transaction,这些actions必须一起成功或一起失败

{
  "expiration": "...",
  "region": 0,
  "ref_block_num": ...,
  "ref_block_prefix": ...,
  "net_usage_words": ..,
  "kcpu_usage": ..,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "...",
      "name": "...",
      "authorization": [{
          "actor": "...",
          "permission": "..."
        }
      ],
      "data": "..."
    }, {
      "account": "...",
      "name": "...",
      "authorization": [{
          "actor": "...",
          "permission": "..."
        }
      ],
      "data": "..."
    }
  ],
  "signatures": [
    ""
  ],
  "context_free_data": []
}

上下文无关actions

action 名称限制

action类型实际上是base32编码的64位整数。这意味着名称前12个字符限制在字符 a-z, 1-5, . 。如果有第13个字符，限制在编码表前16个字符。

transaction确认

transaction完成时，生成一个transaction 收据。收据以hash形式呈现。收到transaction hash不意味着transaction被应答，仅仅意味着节点正确接受了它，
同时也意味着其它生产者会高概率接受它。

通过确认，你应该能看见tranaction历史中的transaction及保护的块号。

action处理器和 action "应用“ 上下文

智能合约提供action 处理器来完成请求的action的工作（下文更多）。没次一个action运行，例如action 通过运行合约实现中的 apply 方法被“应用”，
EOSFORCEIO 创建一个新的action “应用”上下文，action在其中运行。下图描述了action “应用”上下文的关键元素。

从EOSFORCEIO 区块链的全局视角来看，EOSFORCEIO网络中的每个节点获得一份拷贝并且运行每份合约的每个action。有些节点处理合约的实际工作，而其它节点
处理是为了证明transaction块的有效性。因此合约能判定“它们是谁”，或基本地，它们在什么上下文中运行，这很重要。上下文识别信息在action上下文中提供，
如上图所述，通过 receiver, code, action。 receiver是当前处理action的账户。code 是授权合约的账户。action 是当前运行action的ID。

如上讨论，actions运行在transactions内：如果一个transaction失败，transaction内的所有actions的结果必须被回滚。action上下文的一个关键部分是 Current Transaction Data。
这包括transaction 头，transaction中原始action的有序数组，transaction中上下文无关actions数组，实现合约代码定义的可裁剪的上下文无关数据集合（作为blob数组提供），
以及blob数组的一个完整索引。

在处理action之前，EOSFORCEIO 为action设置干净的工作内存。这是为了存放action工作变量。action的工作内存仅仅对那个action可用，甚至对同transaction中其它actions也可用。 
当另一个ction执行时可能会设置变量，但是在另一个action上下文中不可用。在action间传递状态的唯一方法是通过持久化到EOSFORCEIO 数据库中然后从中检索。使用EOSFORCEIO
持久化服务的详情参见 Persistence API 。

action有很多侧面效果。以下是其中一些：


    改变持久化在EOSFORCEIO持久化存储中的状态
    通知接受者当前 transaction
    发送inline action请求给新的接收者
    生成新的（deferred）transactions
    取消现存的deferred transactions(例如取消已经提交的deferred transaction请求)

transaction 限制

每个transaction必须在30毫秒或更少的时间内执行完。如果一个 transaction 包含几个 actions, 这些 actions 执行时间之和超过30毫秒，整个transaction 将失败。
对actions没有并发需求的场景下，这可以通过将耗CPU actions包含在单独的 transaction 中来规避。

# ABI 宏及应用

每个智能合约必须提供一个 apply action 处理器。apply action 处理器是一个函数，监听进入的 actions 及执行想要的动作。为了响应一个特定的 action, 需要 code
来识别和响应特定的 actions 请求。apply 使用 receiver, code 和 action 输入参数作为过滤器来映射到实现特定希望的实现特定actions 的函数。apply 函数能够像下面这样通过
code 参数来过滤：

if (code == N(${contract_name}) {
   // your handler to respond to particular code
}

在某个给定的 code 下，可以通过过滤 action 参数来响应特定的 action 。这通常和 code 过滤结合使用。

if (action == N(${action_name}) {
    //your handler to respond to a particular action
}

EOSIO_ABI 宏

为简化合约开发者工作，EOSIO_ABI 宏封装了 apply 函数底层 action 映射细节，使开发者能专注于应用实现。

#define EOSIO_ABI( TYPE, MEMBERS ) \
extern "C" { \
   void apply( uint64_t receiver, uint64_t code, uint64_t action ) { \
      auto self = receiver; \
      if( code == self ) { \
         TYPE thiscontract( self ); \
         switch( action ) { \
            EOSIO_API( TYPE, MEMBERS ) \
         } \
         /* does not allow destructor of thiscontract to run: eosio_exit(0); */ \
      } \
   } \
} \

开发者仅需指定 code 和 action 来自合约的名字， 所有 C 代码映射逻辑由宏自动生成。宏使用的例子参加上文，例如 EOSIO_ABI( hello, (hi) )， 
hello 和 hi 来自合约。

这个例子可以看到有一个函数 apply 。 它做的所有的事就是登记递交的 actions, 没有其它检查。任何人任何时候可以递交任何 action, 只要块生产者允许。
缺乏任何需要的签名，合约将因消耗带宽而被记账。


apply

apply 是 action 处理器，它监听所有进入的 actions, 并且根据函数内的规格响应。apply 函数需要两个输入参数，code 和 action。


code 过滤

为了响应特定的 action， 结构化 apply 函数如下。你也可以省略 code 过滤构建通用 actions 响应。

if (code == N(${contract_name}) {
    // your handler to respond to particular code
}

也可以在代码块中定义单个 actions 的响应。


action 过滤

为了响应特定的 action, 结构化 apply 函数如下。这通常和 code 过滤结合使用。

if (action == N(${action_name}) {
    //your handler to respond to a particular action
}


wast

任何部署到 EOSFORCEIO 区块链上的程序必须编译成 WASM 格式。这是区块链接受的唯一格式。

一旦 CPP 文件完成，你可以用eosiocpp 工具将它编译成 WASM 的文本版本（.wast）。
eosiocpp 从v1.2.0 版本起不推荐用，v1.3.0 版本将被移除。将被替换成 eosio.wasmsdk 库中的 eosio-cpp 。参数可能也将相应改变。

$ eosio-cpp -o ${contract}.wast ${contract}.cpp


abi

Application Binary Interface(ABI) 是一种基于 JSON 的描述，描述了怎样在 JSON 和 二进制表示 间转换用户 actions。ABI 也描述了
怎样转换数据库状态到/从 JSON。一旦你通过 ABI 描述了合约，开发者和用户将能通过 JSON 无缝和合约交互。

ABI 文件能使用 eosio-cpp 工具传递 --abigen 从 .hpp 文件生成。

$ eosio-cpp -o ${contract}.wast ${contract}.cpp --abigen

下面是一个骨架合约 ABI 示例：

{
  "types": [{
      "new_type_name": "account_name",
      "type": "name"
    }
  ],
  "structs": [{
      "name": "transfer",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "quantity": "uint64"
      }
    },{
      "name": "account",
      "base": "",
      "fields": {
        "account": "name",
        "balance": "uint64"
      }
    }
  ],
  "actions": [{
      "action": "transfer",
      "type": "transfer"
    }
  ],
  "tables": [{
      "table": "account",
      "type": "account",
      "index_type": "i64",
      "key_names" : ["account"],
      "key_types" : ["name"]
    }
  ]
}

你将注意到这个 ABI 定义了一个类型为 transfer 的 action transfer。这告诉 EOSFORCEIO 当看见 ${account}->transfer 时负载是 transfer 类型。
transfer 类型定义在 structs 数组中 name 为 transfer 的对象中。

  "structs": [{
      "name": "transfer",
      "base": "",
      "fields": {
        "from": "account_name",
        "to": "account_name",
        "quantity": "uint64"
      }
    },{
...

ABI 有好几个字段，包括 from, to, 和 quantity。这些字段有对应的类型 account_name 和 uint64。account_name 是内建类型，将字符串表示成base32编码的 uint64。
了解更多关于内建类型，点这里。https://github.com/EOSIO/eos/blob/master/libraries/chain/abi_serializer.cpp#L65-L103

# 多索引数据库编程接口



# 命名规范



# C++/C 编程接口