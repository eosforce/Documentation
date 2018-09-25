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

EOSFORCEIO支持两种基本通信模型，inline 和 deferred。





# ABI 宏及应用



# 多索引数据库编程接口



# 命名规范



# C++/C 编程接口