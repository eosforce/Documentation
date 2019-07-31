# Background

The Crypto economy has ushered in a critical phase from a social experiment to a mass commercial implementation.

A mass commercial implementation of crypto economy which means a huge trading pressure. To efficiently carry a huge amount of trading demand, a blockchain system must provide strong performance firstly. To achieve this, you need set higher requirements on the whole node, such as configuring better hardware machines, larger RAM, more stable network, faster NET, lower latency, etc. Obviously, the high threshold of whole node will result in a decrease in the amount of stable Block Producers (BPs). If the POS mechanism is adopted in such a blockchain system, the system will quickly converge to centralized. In order to strike a balance between the high performance and decentralization, the DPOS consensus algorithm is undoubtedly the best choice at the moment, as well as the best settlement for governing small quantity of nodes.

The EOSIO based on the DPOS consensus algorithm came into being, which brings the first light of the mass commercial implementation of crypto economy to crypto community. The fully  effectiveness of election mechanism is the key to the survival of DPOS and is also related to whether DPOS will lead the crypto trend after POW.

In order to accelerate the mass commercial implementation of crypto economy, the EOSC community has optimized the election mechanism of EOSIO and launched the EOSC mainnet at the genesis block height of one. Through persistently upgrading EOSC mainnet that makes EOSC Network evolve toward the decentralized high-performance smart contract platform.



# Consensus Mechanism

EOSC utilizes the consensus mechanism of EOSIO, which is DPOS BFT Pipeline Consensus. EOSC enables blocks to be produced every 3 seconds with no consecutive blocks, which is different from EOSIO'S model of producing blocks every 0.5 second with 6 consecutive blocks produced by 1 node.  Even though consecutive block-generating can reduce the waiting time of unpacked trades, it may affect the chain stability because of the unsatisfactory network environment, resulting in a large number of microforks.

Currently, the consensus mechanism of EOSIO is not to be perfect. However, as a DAPP platform, the confirmation time of block is not the first optimization task of the chain. EOSC must consider the consensus mechanism in a high-load environment. In the current situation that the parallel computing mechanism is not refined, a hasty improvement of pipelined confirmation mechanism may cause big problems.

In the future, the consensus mechanism of EOSC will evolve in parallel from two directions:

1.To develop and update the consensus mechanism compatible with EOSIO. Based on the current development process of EOSIO, we estimate that after the parallel improvement, EOSIO will upgrade the consensus algorithm to achieve faster block confirmation time. 

2.To supplement the current DPOS consensus adapted to other confirmation-based consensus mechanisms. In one respect, it will achieve the interaction of the embedded Layer 2 chain consensus and the main chain. On the other respect, it can achieve a more decentralized cross-chain mechanism with other consensus mechanisms.



# EOSC Technology Improvement



## Fee-based Resource Model

EOSIO's payment model of CPU and NET is a good design from the aspect of technology but is too complicated for users. Meanwhile, it does not prompt DAPP developers to optimize their contracts. On the other hand, the purchase method of RAM by EOSIO will lead to some hoarding behaviors, which is not conducive for the development of DAPP ecology. Therefore, EOSC innovatively designed a new resource model, through the optimization in practice, exploring the fee-based resource model in the complex smart contract environment, and solving the resource problem of EOS ecology completely.

Firstly, CPU and NET resources consumed by users are paid by EOSC with fee model. Developers can set a fee for the defined Action in DAPP, which enables system to control the resource usage of Action. On the one hand, it is easy for users to understand resource consumptions. On the other hand, it also strongly prompts DAPP developers to optimize the usage of contract resources, so that the DAPP ecology can develop well.

EOSC allocates RAM resource in the way similar to renting Cloud Hosting. Users can pay the lease of RAM resources by using voting rewards, which will keep users away from worries of rent payment as well as rent arrears. EOSC can effectively avoid speculative acts of RAM resource by adopting. Through “sale by lease “method, EOSC can effectively avoid speculation against RAM resources, so that the of development of DAAP will not be affected by RAM price, which will effectively be promoting DAPP ecological construction.

In addition to exploring new resource models, EOSC has been working on exploring the resource model which is compatible with EOSIO in terms of mechanism. For CPU and NET resources, users can pay fee based on rewarding age to receive CPU and NET resources like the depositing mechanism of EOSIO. For RAM, users can vote and deposit to receive RAM resources like the purchasing mechanism based on the market of EOSIO. In this way, DAPP developers can quickly cut into EOSC from other EOSIO chains and smoothly shift to the resource model of EOSC.



## Smooth Update Mechanism

The election mechanism of EOSC prompt Block Producers to actively participate in advancing technology upgrades. Unlike the split of the EOSIO community's node versions, EOSC actively promotes technology upgrades and updates in practice.

In order to achieve a smoother incompatible upgrade process, EOSC added an update mechanism based on the effective block heights. The community can confirm the effective block height of a function by multi-signing, achieving a decentralized smooth upgrade process. Compared to the label solution based on blockchain extended data put forward by EOSIO recently，the upgrade mechanism of EOSC is more friendly and understandable. EOSC is the first to achieve the decentralized “soft fork” upgrade process, which is the fundamental for EOSC to keep evolving to solve all kinds of mechanism problems.

On the other hand, the function of setting chain property based on multi-sign can provide with the community with a scheme of decentralized chain setting, various parameters and configurations can be modified in a decentralized way according to the actual development, making the community develop better.



## Node Heartbeat Mechanism and Stable Block-Generating Interval

In order to promote the stability of mainnet, EOSC strengthened the construction of Block Producer (BP) candidates from the aspect of Economy Model. Meanwhile, EOSC adds the node Heartbeat Mechanism on the chain to prompt nodes to strengthen their stability, making the mainnet more stable.

On the basis of Heartbeat Mechanism, EOSC can confirm the operation status of nodes, punish the faulty nodes on the chain, and further supervise the constructions of nodes to prevent the instability of mainnet.

At the beginning of the launch, block-generating interval was extended to prevent the occasional soft forks with incomplete network infrastructure. The mechanism designed by EOSIO is that half-second of block-generating interval and 6 consecutive blocks produced by one node, which can improve the availability of the chain in the future. However, it is not applicable in the current network environment. EOSC extended the block-generating interval with a practical attitude, will accelerate block generating to effectively avoid soft forks. Meanwhile, the reduction of block amount can greatly increase the synchronization rate of whole node, so that there will be more full nodes and the availability of whole network will be enhanced.



## More Contract Layer API

In order to make it more convenient for DAPP developers to develop the contracts, some APIs have been added, and some specific adjustments have been made to the system contracts.

Firstly, API for obtaining the height of the block has been added, which enables developers to get current block heights more easily and effectively. Based on this API, the contract can effectively avoid attacks from clogged blocks and retry activities. Secondly, API for obtaining chain setting information has been added, which enables developers can adapt parameter corrections and upgrades of the chain on the contract layer, so the contract can also smoothly follow the chain smoothly. Finally, independent core token contract was utilized before starting the chain to avoid attacks from fake tokens, which enables users to distinguish attacks from fake tokens clearly.



## Cross-Chain Service Adaption

At the beginning of the launch, the EOSC Network team forecasted that the support to cross-chain would be the basic function of the public chain. Therefore, the EOSC team started the development of Codex project that supporting Relay service to the various chains by building Codex. Relay, In this way, the cross-chain mechanism between the various chains can be implemented. Codex.Relay is more completely supported. Through mutual operations by BPs on two chains, the complete Cross-chain Mechanism will be achieved that the degree of decentralization of any chain will not be reduced during the process.

Great extensibility can be obtained through Cross-chain Mechanism. Based on the relay service, Layer 2 sub-chain can be added. The services and DAPPs with large resource consumption can be operated based on sub-chain, and the computing result or the core state can be synchronized by the relay service, so that the specified subchains for storage, computing, DAPP, and random numbers can be added later, thereby expanding the function.



## Highly-customizable EOSIO Blockchain Development Framework

Based on the relay service, Layer 2 sub-chain can be added. In the future, all kinds of subchains will play a big role in EOSIO ecology. However, it should be noted that there is a high threshold for developing a customizable blockchain project based on EOSIO. For this reason, EOSC Network team enabled the Codex.io project to reduce the development threshold of subchains, which is a highly customizable EOSIO blockchain development framework, providing developers with a more economical and friendly subchain development experience.

The EOSC Network team has accumulated a lot of experiences based on the EOSIO development blockchain in the development, and hopes to bring these experiences to their maximum value. Codex.io is an EOSIO blockchain development framework that works out of the box. The developers can quickly launch a chain with Codex.io. After simple configuration, the developer can customize all kinds of symbols and select economy system and resource model freely. On this basis, developers only need to pay attention to the problems of the chain itself. According to this, it can be implemented based on the contract or chain native layer. Codex.io can facilitate developers to expand in the native layer of the chain, solving some performance issues as well as enhancing the function of the chain greatly.

Codex.io integrates most of the functions put forward by EOSIO. With an inclusive attitude, Codex.io allows developers to freely combine functions on the chain: including the subsistence allowance system, account system, various black and white list mechanisms, common governance mechanisms and voting mechanisms, as well as various plugins.

With Codex.io, a large amount of Layer 2 subchains will be integrated in the future, which will provide unlimited scalability.



# Economic Model & Resource Model



## Token Issue

The initial circulation of EOSC was 1 billion, and the initial asset allocation was completed according to the mapping snapshot of EOS ERC20 on June 3, 2018 (BJT). Users can activate the EOSC account assets by offline signing with the mapped EOSIO private key. 



## Resource Usage

The key function of EOSC token is to obtain resources on the chain. Each transaction on EOSC mainnet consumes certain resources. There are 3 broad classes of resources:

1． Bandwidth resource (NET)

2． Computing resource (CPU)

3． Storage resource (RAM)

NET and CPU are recoverable resources, which can provide the network bandwidth and computing resources consumed by transferring and implementing smart contract. Specifically, when one transaction is packed and synchronized to the network by block producers, NET resource will be consumed. The more data generated by operations; the more NET resources will be consumed. And when smart contract is called, CPU computing resources will be consumed by the execution of the contract code loaded into memory. The longer the contract is executed, the more CPU resources will be consumed.

 RAM is used to store information such as account information and smart contract status. The more data need to be stored on blockchain, the more RAM resource is required. Unlike NET and CPU, RAM resources need to be occupied for a long time. 

EOSC mainnet designed a fee-based resource model to avoid waste or abuse of limited resources on the chain, which is both user-friendly and developer-friendly. Users can obtain NET and CPU resources by paying fees and obtain RAM resources by voting and depositing.



## Block Rewards

EOSC mainnet enables blocks to be produced every 3 seconds. Every time when there is a new block produced, the system will increase a certain amount of EOSC, called Block Rewards. Block Rewards are distributed as follows:

\1. 10% of block rewards are distributed by the block-generating nodes in proportion to the number of blocks.

\2. 10% of block rewards are distributed by all profitable nodes' rewards according to the proportion of voting weighted votes.

\3. For 50% of block rewards, the nodes can freely adjust the dividend ratio.

\4. 30% of block rewards goes to decentralized budget system.

 

The inflation rate of EOSC is 20% of the total amount of the previous year's token, which means that the total amount of annual block reward is 20% of all EOSC in previous year's system.

 

# Community Governance



## Time-Weighted Election Mechanism

EOSC Network through continuously improving election mechanism to achieve the decentralization of DPOS consensus mechanism.



### Weighted Votes

The ranking of nodes is determined by the number of weighted votes polled. The more weighted votes polled by the node, the higher the ranking of the node.

The way to participating in EOSFORCE mainnet voting can choose the current voting and the fix-time voting. The time periods can be selected in fix-time voting are the block heights corresponding to 3 months, 6 months, 12 months and 24 months respectively. The tokens are locked during the fix-time voting period and automatically unlocked when it expires.

There are no fixed periods in current voting. The voting tokens can be redeemed at any time. The redemption needs 3 days to unlock the block height.

In addition, the voting weights vary according to different time periods. Weight of current voting is 0.2, while 3 months, 6 months, 12 months, 24 months correspond to different weights of 1, 2, 4 and 8 months respectively.

Voters can switch nodes at any time in both Fix-time voting and current voting, just paying the fees of the system itself with no waiting time.



### 1-Token-1-Vote

The election mechanism determines whether DPOS will be fully effective.

EOSC mainnet utilizes the vote mechanism of 1-token-1-vote, so that one account can vote to multiple nodes, but each EOSC can only vote for one node at a time.  EOSC mainnet proves in practice that 1-token-1-vote can effectively avoid node alliances, building a more equitable, sensible and vibrant room for node election.

Through the election mechanism of 1-token-1-node, no node is able to occupy long-term positions of BP, and there is enough room for new nodes to become BPs. The election mechanism of EOSC mainnet is imbued with vibrancy.



## Decentralized Protocol Governance

The open protocol governance mechanism of EOSC Network enables Block producers (BP) to update network through the governance on the chain, ensuring that the development of EOSC protocol follows the volition of community.



### Voting Reward

Voting reward is an incentive method of EOSC mainnet to encourage users on the chain actively participate in the voting. By voting, token holders can receive rewards which come from tokens increased annually. 

The amount of voting rewards that a voter can receive depends on the proportion of his weighted votes in total votes. The higher proportion of weighted votes is, the higher voting rewards can be received.

Rewards of fix-time voting, and current voting can be received and re-invested at any time, but the principal of fix-time voting cannot be released before the end of the fix-time period.



### Node Punishment

Nodes of EOSC mainnet are divided as BP and profitable node.

Block-generating node, which indicates block producer (BP), is one of the top 23 nodes based on the weighted votes ranking.

The profitable node is a registered node that can obtain the rewards of voting. Anyone can access the EOSC mainnet to become a profitable node and share 10% of block reward according to the proportion of weighted votes.

DPOS-based blockchain system need to be operated and maintained by multiple BPs. If there are BPs fault or do evil, the security and stability of the system will be threatened. EOSC mainnet established a Node Punishment Mechanism for the purpose of ensuring the security and stability of the system more effectively, eliminating evil nodes and passive nodes, electing the nodes that can truly serve the community and support the system operation.

Node Punishment Mechanism is mainly embodied in three aspects: 1.  To reduce the block reward of block producers; 2. To deduct the deposit of block producers; 3. To lock up nodes.



#### BP Reward

According to rules of block rewards distributing, block producers can receive 10% of block rewards, which are not distributed by block producers, but by the weight of block-generating age (the continuous time of block-generating).

Specifically, EOSC mainnet put forward the concept of “Block producers Stability”. The initial value of BP stability is 0. If there is no block missing within each cycle of 600 blocks, 1 will be added. The maximum is 3800. There will be no value added if it reaches 3800. Once there is block missing, the value will be reset to 0.

The Stability of Block Producer is related to the weight of BP rewards. The weight will be higher if blocking-generating is persistent and stable. In the case of generating same blocks, the higher stability the more node rewards.



#### BP Deposit

The node needs to provide a part of tokens as a deposit to become BP. The deposit will be deducted for the block missing of BP, and twice the deposit of the reward of the block will be deducted for each missed block. If the deposit is under the minimum amount set by the system, there will be no block-generating reward, and the node will not be allowed to join the election of BP.

If the deposit is deducted to a minus when generating blocks and no supplement is offered, there will be no reward for all users voted to this node and no income for the node.



#### Node Locking

 After the node has continuously missed more than 9 blocks, anyone can initiate the multi-sig proposal to punish the frozen node. The penalty will be divided by the user who proposed the multi-sig punishment and top 16 block-generating nodes which agree with the multi-sig punishment. (20% for initiator, agreed to sign more than 80%). A deposit of 100 EOSC is required for the proposal of multi-sig punishment. After 14400 block heights, the deposited token can be returned whether the proposal is adopted or not.

 The frozen block node enters one-hour observation period during which user voting without mining reward, and nodes do not have any reward neither. After the observation period, BP can apply for a normal status to participate in the normal election and make normal mining profits.



## Decentralized Budget System on The Chain

The decentralized Budget System on the chain of EOSC mainnet provides budget support to community contributors and developers that anyone can initiate a budget proposal and will be approved by the decentralized budget management committee. The long-term sustainable development of blockchain can be achieved.

 Rules of budget proposal are as follows:

1. The sponsor nees to pledge 100 EOSC as deposit for the proposal to prevent bad proposals. The deposit will be returned after the proposal is passed.

2. The proposer needs to specify the objectives of the proposal, the implementation details (cycle, cost, amount) and the return that the proposal will bring.

3. After the proposal is approved by the management committee, there will be a 14-day publicity period. If more than one-third of the profitable nodes are opposed during the publicity period, the proposal can be rejected. If the proposal is not rejected during the publicity period, the system will automatically issue the award to the applicant's account on a monthly basis after the publicity period.

4. After the proposal has passed the publicity period, the proposer will get the EOSC tokens applied for in the next block award to implement the proposal content.

5. During the reward payment period, the management committee and the profitable nodes are greater than one-third of the veto, and the proposal award can be suspended.
