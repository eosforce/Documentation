# EOS源码备忘-Block Produce机制

---------------------------------------

这里我们主要讨论EOS出块相关代码，在EOS中由BP节点出块，这些区块构成了一条链，这里的逻辑Eosforce和Eos没有太大区别。

## 1.总体流程

nodeos启动时会加载producer_plugin，在这个插件的start函数中会调用schedule_production_loop()开始循环：

```cpp
void producer_plugin::plugin_startup()
{ try {
   // ...

   // 这里启动出块循环，在循环中通过回调schedule_production_loop的形式loop
   my->schedule_production_loop();

   ilog("producer plugin:  plugin_startup() end");
} FC_CAPTURE_AND_RETHROW() }
```

schedule_production_loop()中会调用`producer_plugin_impl::start_block(bool &last_block)`尝试出块，根据出块尝试结果，会执行maybe_produce_block()或者等待。

start_block()会先判断节点是否应该出块，如果节点轮到出块，则会调用 `chain.start_block(block_time, blocks_to_confirm)`，这里最终调用chain/controller.cpp中的`start_block`函数出块，注意代码中有很多个start_block函数。

chain/controller.cpp中的`start_block`函数主要是设置好_pending_block_state，之后执行on_block transaction。on_block是一个很基础的合约，在eosio.system下的 `system_contract::onblock`，主要是添加出块奖励和重选出块节点，注意这里eosforce实现了不同的onblock合约。

准备好出块之后，在`producer_plugin_impl::start_block`中会执行需要执行的trx。

计算好区块之后，逻辑回到`schedule_production_loop`，根据start_block的结果，如果成功，会调用`maybe_produce_block`，这里最终调用`void producer_plugin_impl::produce_block()`，为待出的块计算签名，广播，最后调用commit_block出发新区块的处理逻辑。

下面详细分析一下以上的流程。

## 2.几个问题

在分析代码之前，先要看一下nodeos的几个问题。

EOS开发者在实现出块逻辑时，代码里有大量的关于`_pending_block_mode`的if判断，我们这里先梳理一下`_pending_block_mode`不同值在不同时刻的意义，以便后面不会产生混乱。

另外一方面，在最近的几个版本中，为了解决同步太慢的问题，开发者对同步节点逻辑做了一定的特化，减少了很多被认为是无用的计算，虽然我们这里讨论的是出块逻辑，但是很多地方，同步和出块共用了一些代码，在这些地方也是用了大量的if判断来区分逻辑，这里也先梳理一下意义。

### 2.1 `_pending_block_mode`

`_pending_block_mode` 在 https://github.com/EOSIO/eos/issues/3161 中加入，
完整的提交在 https://github.com/EOSIO/eos/pull/3170 。

这个修改改动不小，改变了之前的出块代码结构。

提交是为了解决下面这个bug：

```
Upon receiving a transaction that throws a soft-fail excepion, which indicates the transaction may be good but the block is full, the producer plugin assumes it is the next active producer and attempts to sign and send off a block, expects success and then reapplies the transaction.

If any of the following assumptions fail this results in a relatively tight loop of creating a block which is exhausted, attempting to play a transaction which soft fails, dropping that block on the floor and trying again until the transaction hard fails due to its own expiration.

This is obviously a bug
```

在之前的版本中有一个当执行transaction时抛出了一个soft-fail异常时，整个出块插件会不停循环调用loop的bug，在原来的实现当某个transaction soft失败之后，出块节点会认为区块中有个空位置，此时会重新跑一边loop来尝试将未被apply的transaction加入区块，但是，如果由于区块中transaction已满而造成错误时，系统会不断运行loop，因为第二次运行依然是满的，直到有些交易因为超时而被抛弃的时候，此时才能正常出块。

在修正这个bug的同时，EOS团队重构了部分代码，将之前的一些分散处理的错误情况改为记录`_pending_block_mode`状态，原本`block_production_loop()`有这些返回：

```cpp
namespace block_production_condition {
   enum block_production_condition_enum
   {
      produced = 0,
      not_synced = 1,
      not_my_turn = 2,
      not_time_yet = 3,
      no_private_key = 4,
      low_participation = 5,
      lag = 6,
      exception_producing_block = 7,
      fork_below_watermark = 8,
   };
}
```

通过这些返回来判断分别执行现在用`_pending_block_mode`判断的逻辑。很多原本在之前的`maybe_produce_block`（后来改名现在是`produce_block`）判定的逻辑挪到在start_block中判断当前状态：

```cpp

...

    // 默认是producing，即正常出块
   _pending_block_mode = pending_block_mode::producing;

   // If the next block production opportunity is in the present or future, we're synced.
   if( !_production_enabled ) {
       // 对应not_synced，当前没有同步完成
      _pending_block_mode = pending_block_mode::speculating;
   }

   // Not our turn
   const auto& scheduled_producer = hbs->get_scheduled_producer(block_time);
   if( _producers.find(scheduled_producer.producer_name) != _producers.end()) {
       // 对应not_my_turn，不是轮到当前节点出块
      _pending_block_mode = pending_block_mode::speculating;
   }

   auto private_key_itr = _private_keys.find( scheduled_producer.block_signing_key );
   if( private_key_itr == _private_keys.end() ) {
      ilog("Not producing block because I don't have the private key for ${scheduled_key}", ("scheduled_key", scheduled_producer.block_signing_key));
      // 对应no_private_key 没有块签名私钥
      _pending_block_mode = pending_block_mode::speculating;
   }

   // determine if our watermark excludes us from producing at this point
   auto currrent_watermark_itr = _producer_watermarks.find(scheduled_producer.producer_name);
   if (currrent_watermark_itr != _producer_watermarks.end()) {
      if (currrent_watermark_itr->second >= hbs->block_num + 1) {
         elog( "Not producing block because \"${producer}\" signed a BFT confirmation OR block at a higher block number (${watermark}) than the current fork's head (${head_block_num})",
             ("producer", scheduled_producer.producer_name)
             ("watermark",currrent_watermark_itr->second)
             ("head_block_num",hbs->block_num));
             // 对应fork_below_watermark 被watermark排除出块
         _pending_block_mode = pending_block_mode::speculating;
      }
   }
...
```

## 3.出块判定 producer_plugin_impl::schedule_production_loop()



## 4.块计算逻辑 producer_plugin_impl::start_block()

## 5.出块逻辑 producer_plugin_impl::maybe_produce_block()

## 6.需要留意的问题