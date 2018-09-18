# 关于EOS与Eosforce源码备忘系列文章的说明

后续一段时间，原力团队（https://www.eosforce.io/）将会陆续发表一系列EOS与EosForce源码分析的文章，
这些文章将会发表在原力的文档系统（https://eosforce.github.io/Documentation/#/zh-cn/code/block_produce）和博客（https://eosforce.io/blog/index.html#/cn）上，

现在计划目录如下：

- Block Produce流程代码分析
- Push Transaction流程代码分析
- EOS共识机制与源码分析
- EosForce系统合约分析
- EOS系统合约分析
- EOS与EosForce区别整理
- Plugin开发备忘：以MongoDB Plugin为例
- 如何构建EOS的单元测试
- WASM虚拟机相关整理
- EOS与EosForce中的C++异常

后续也会根据需求添加新的文档，
之所以原力团队要整理这一系列文档，主要出于以下原因：

- 第一，目前网上相关分析不是太粗略就是太陈旧，无论是为了EOS或是EosForce社区的发展，都需要这方面文档让大家更好的了解EOS与EosForce。
- 第二，我们认为整个EOS或EosForce的发展取决于DApp生态的建立，而开发DApp需要对EOS底层技术有深入的了解，我们希望这一系列文章可以推进DApp生态的发展。
- 第三，对于原力团队，我们一直基于EOS开发我们的EosForce项目，完善这方面文档，可以加快团队的建设，减少成员的学习成本，另一方面，通过逐行的分析EOS项目源码，可以让团队对于EOS项目理解更加深刻。
- 第四，EOS源码实现时注释和封装较少，很多函数实现较为冗长，一份详细的文档可以作为后续开发的备忘。

出于以上原因原力团队将会在接下来一段时间利用开发工作的间隙完成这些文章，很多分析因为是出于备忘的目的，可能会比较冗长，通读可能会比较枯燥，比较适合阅读代码时的参考。

EOS是一个发展迅速的项目，对于原力团队，必然会有一些理解代码过程中的错误，欢迎大家提出来，如果大家有一些疑问也可以联系原力团队：

加微信号：EOSforce