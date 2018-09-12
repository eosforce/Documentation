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

## 3.块计算逻辑 producer_plugin_impl::start_block()

首先看下 persisted transactions:

```cpp
      // 这里处理持久化交易, 所有过期的交易会被忽略.
      // remove all persisted transactions that have now expired
      auto& persisted_by_id = _persistent_transactions.get<by_id>();
      auto& persisted_by_expiry = _persistent_transactions.get<by_expiry>();
      while(!persisted_by_expiry.empty() && persisted_by_expiry.begin()->expiry <= pbs->header.timestamp.to_time_point()) {
         persisted_by_expiry.erase(persisted_by_expiry.begin());
      }
```

这里的持久化交易是对于所谓的`RPC Node`来讲的,`RPC Node`作为直接面向某些用户,负责接收用户的请求,
对于这类节点,发送给它的交易都应该执行,具体的内容可以参考 https://github.com/EOSIO/eos/issues/3295 中的描述.

下面是处理所有没有被执行的交易.

如果一个节点既不是出块节点,也不是RPC节点的话,那它是不需要执行任何交易的:

```cpp
         if (_producers.empty() && persisted_by_id.empty()) {
            // if this node can never produce and has no persisted transactions,
            // there is no need for unapplied transactions they can be dropped
            chain.drop_all_unapplied_transactions();
         } else {
            ...
         }
```

之后筛选出所有要被执行的交易:

```cpp
            std::vector<transaction_metadata_ptr> apply_trxs;
            { // derive appliable transactions from unapplied_transactions and drop droppable transactions
               auto unapplied_trxs = chain.get_unapplied_transactions();
               apply_trxs.reserve(unapplied_trxs.size());

               // 这个函数判断要执行trx的类型
               auto calculate_transaction_category = [&](const transaction_metadata_ptr& trx) {
                  if (trx->packed_trx.expiration() < pbs->header.timestamp.to_time_point()) {
                     // 已经超时,会被忽略
                     return tx_category::EXPIRED;
                  } else if (persisted_by_id.find(trx->id) != persisted_by_id.end()) {
                     // 持久化交易,会执行
                     return tx_category::PERSISTED;
                  } else {
                     // 常规交易
                     return tx_category::UNEXPIRED_UNPERSISTED;
                  }
               };

               for (auto& trx: unapplied_trxs) {
                  auto category = calculate_transaction_category(trx);
                  // 一个交易被执行条件:
                  //   1. 没有过期
                  //   2. 当前节点出块 或者 当前节点不出块,但交易是持久交易
                  if (category == tx_category::EXPIRED
                  || (category == tx_category::UNEXPIRED_UNPERSISTED && _producers.empty())) {
                     chain.drop_unapplied_transaction(trx);
                  } else if (category == tx_category::PERSISTED
                  || (category == tx_category::UNEXPIRED_UNPERSISTED
                        && _pending_block_mode == pending_block_mode::producing)) {
                     apply_trxs.emplace_back(std::move(trx));
                  }
               }
            }
```

这里会把所有要执行的交易存入`apply_trxs`.下面会逐个执行交易:

```cpp
            for (const auto& trx: apply_trxs) {
               // 判断是否超出了最大的执行时间,注意在EOS中每个块的cpu执行时间是有上限的.
               if (block_time <= fc::time_point::now()) exhausted = true;
               if (exhausted) {
                  break;
               }

               try {
                  // 刷新时间记录
                  auto deadline = fc::time_point::now() + fc::milliseconds(_max_transaction_time_ms);
                  bool deadline_is_subjective = false;
                  if (_max_transaction_time_ms < 0 || (_pending_block_mode == pending_block_mode::producing && block_time < deadline)) {
                     deadline_is_subjective = true;
                     deadline = block_time;
                  }

                  // 执行交易,这里的过程我们在其他文档中具体分析
                  auto trace = chain.push_transaction(trx, deadline);
                  if (trace->except) {
                     if (failure_is_subjective(*trace->except, deadline_is_subjective)) {
                        // 根据trx执行来判断是否超出了最大的执行时间
                        exhausted = true;
                     } else {
                        // this failed our configured maximum transaction time, we don't want to replay it
                        chain.drop_unapplied_transaction(trx);
                     }
                  }
               } catch ( const guard_exception& e ) {
                  app().get_plugin<chain_plugin>().handle_guard_exception(e);
                  return start_block_result::failed;
               } FC_LOG_AND_DROP();
            }
```

这里注意`guard_exception`. TODO

以上代码执行了所有的交易,下面是对延迟交易的处理:

注意这里所谓的黑名单交易,这里和节点的黑白名单功能不同,这里是用来排除所有执行失败的延迟交易.

```cpp
         if (_pending_block_mode == pending_block_mode::producing) {

            // 整理_blacklisted_transactions, 如果已经超时,则删去交易(此时肯定是执行不了了)
            auto& blacklist_by_id = _blacklisted_transactions.get<by_id>();
            auto& blacklist_by_expiry = _blacklisted_transactions.get<by_expiry>();
            auto now = fc::time_point::now();
            while (!blacklist_by_expiry.empty() && blacklist_by_expiry.begin()->expiry <= now) {
               blacklist_by_expiry.erase(blacklist_by_expiry.begin());
            }

            // 执行延迟交易
            auto scheduled_trxs = chain.get_scheduled_transactions();

            for (const auto& trx : scheduled_trxs) {
                // 同样的逻辑,没有抽象
               if (block_time <= fc::time_point::now()) exhausted = true;
               if (exhausted) {
                  break;
               }

               // configurable ratio of incoming txns vs deferred txns
               // 这是一个新近交易和延迟交易执行的比例,
               // 注意当trx及其多时,需要平衡两种交易的执行时间
               while (_incoming_trx_weight >= 1.0 && orig_pending_txn_size && _pending_incoming_transactions.size()) {
                  auto e = _pending_incoming_transactions.front();
                  _pending_incoming_transactions.pop_front();
                  --orig_pending_txn_size;
                  _incoming_trx_weight -= 1.0;
                  on_incoming_transaction_async(std::get<0>(e), std::get<1>(e), std::get<2>(e));
               }

               if (block_time <= fc::time_point::now()) {
                  exhausted = true;
                  break;
               }

               // 如果之前执行出错过,就不执行了
               if (blacklist_by_id.find(trx) != blacklist_by_id.end()) {
                  continue;
               }

               // 以下的逻辑差不多
               try {
                  auto deadline = fc::time_point::now() + fc::milliseconds(_max_transaction_time_ms);
                  bool deadline_is_subjective = false;
                  if (_max_transaction_time_ms < 0 || (_pending_block_mode == pending_block_mode::producing && block_time < deadline)) {
                     deadline_is_subjective = true;
                     deadline = block_time;
                  }

                  auto trace = chain.push_scheduled_transaction(trx, deadline);
                  if (trace->except) {
                     if (failure_is_subjective(*trace->except, deadline_is_subjective)) {
                        exhausted = true;
                     } else {
                        auto expiration = fc::time_point::now() + fc::seconds(chain.get_global_properties().configuration.deferred_trx_expiration_window);
                        // this failed our configured maximum transaction time, we don't want to replay it add it to a blacklist
                        _blacklisted_transactions.insert(transaction_id_with_expiry{trx, expiration});
                     }
                  }
               } catch ( const guard_exception& e ) {
                  app().get_plugin<chain_plugin>().handle_guard_exception(e);
                  return start_block_result::failed;
               } FC_LOG_AND_DROP();

               _incoming_trx_weight += _incoming_defer_ratio;
               if (!orig_pending_txn_size) _incoming_trx_weight = 0.0;
            }
         }
```

最后会再检查下执行时间,同时发起对incoming_transaction的处理,如果没问题的话,会返回成功:

```cpp
         if (exhausted || block_time <= fc::time_point::now()) {
            return start_block_result::exhausted;
         } else {
            // attempt to apply any pending incoming transactions
            _incoming_trx_weight = 0.0;
            while (orig_pending_txn_size && _pending_incoming_transactions.size()) {
               auto e = _pending_incoming_transactions.front();
               _pending_incoming_transactions.pop_front();
               --orig_pending_txn_size;
               on_incoming_transaction_async(std::get<0>(e), std::get<1>(e), std::get<2>(e));
               if (block_time <= fc::time_point::now()) return start_block_result::exhausted;
            }
            return start_block_result::succeeded;
         }
```

以上就是`producer_plugin_impl::start_block`的流程,这个过程成功之后,如果是出块节点,此时的pending_block已经计算好了,下一步就是调用`maybe_produce_block`完成出块流程.

## 4.出块逻辑 producer_plugin_impl::maybe_produce_block()

上述start_block执行完毕之后,如果出块正常,则会调用`maybe_produce_block`:

```cpp
      _timer.async_wait([&chain,weak_this,cid=++_timer_corelation_id](const boost::system::error_code& ec) {
         auto self = weak_this.lock();
         if (self && ec != boost::asio::error::operation_aborted && cid == self->_timer_corelation_id) {
            // pending_block_state expected, but can't assert inside async_wait
            auto block_num = chain.pending_block_state() ? chain.pending_block_state()->block_num : 0;
            auto res = self->maybe_produce_block();
            fc_dlog(_log, "Producing Block #${num} returned: ${res}", ("num", block_num)("res", res));
         }
      });
```

让我们来看下`maybe_produce_block`实现:

```cpp
bool producer_plugin_impl::maybe_produce_block() {
   auto reschedule = fc::make_scoped_exit([this]{
      // 完成之后会回调schedule_production_loop 来继续出下一个块
      schedule_production_loop();
   });

   try {
      // 完成出块并广播
      produce_block();
      return true;
   } catch ( const guard_exception& e ) {
      app().get_plugin<chain_plugin>().handle_guard_exception(e);
      return false;
   } catch ( boost::interprocess::bad_alloc& ) {
      raise(SIGUSR1);
      return false;
   } FC_LOG_AND_DROP();

   fc_dlog(_log, "Aborting block due to produce_block error");
   // 清理缓存块的状态
   chain::controller& chain = app().get_plugin<chain_plugin>().chain();
   chain.abort_block();
   return false;
}
```

以下是`produce_block`:

```cpp
void producer_plugin_impl::produce_block() {
   //ilog("produce_block ${t}", ("t", fc::time_point::now())); // for testing _produce_time_offset_us

   // 确定一定是出块状态才能调用到这里
   EOS_ASSERT(_pending_block_mode == pending_block_mode::producing, producer_exception, "called produce_block while not actually producing");

   //检查
   chain::controller& chain = app().get_plugin<chain_plugin>().chain();
   const auto& pbs = chain.pending_block_state();
   const auto& hbs = chain.head_block_state();
   EOS_ASSERT(pbs, missing_pending_block_state, "pending_block_state does not exist but it should, another plugin may have corrupted it");
   auto signature_provider_itr = _signature_providers.find( pbs->block_signing_key );

   EOS_ASSERT(signature_provider_itr != _signature_providers.end(), producer_priv_key_not_found, "Attempting to produce a block for which we don't have the private key");

   //idump( (fc::time_point::now() - chain.pending_block_time()) );
   // 完成出块
   chain.finalize_block();

   // 签名区块
   chain.sign_block( [&]( const digest_type& d ) {
      auto debug_logger = maybe_make_debug_time_logger();
      return signature_provider_itr->second(d);
   } );

    // 把完成的区块广播出去,并调用其他系统的handle函数
   chain.commit_block();
   auto hbt = chain.head_block_time();
   //idump((fc::time_point::now() - hbt));

   block_state_ptr new_bs = chain.head_block_state();
   _producer_watermarks[new_bs->header.producer] = chain.head_block_num();

   ilog("Produced block ${id}... #${n} @ ${t} signed by ${p} [trxs: ${count}, lib: ${lib}, confirmed: ${confs}]",
        ("p",new_bs->header.producer)("id",fc::variant(new_bs->id).as_string().substr(0,16))
        ("n",new_bs->block_num)("t",new_bs->header.timestamp)
        ("count",new_bs->block->transactions.size())("lib",chain.last_irreversible_block_num())("confs", new_bs->header.confirmed));

}
```

在`finalize_block`中会首先更新cpu和net的资源:

```cpp
      // Update resource limits:
      resource_limits.process_account_limit_updates();
      const auto& chain_config = self.get_global_properties().configuration;
      uint32_t max_virtual_mult = 1000;
      uint64_t CPU_TARGET = EOS_PERCENT(chain_config.max_block_cpu_usage, chain_config.target_block_cpu_usage_pct);
      resource_limits.set_block_parameters(
         { CPU_TARGET, chain_config.max_block_cpu_usage, config::block_cpu_usage_average_window_ms / config::block_interval_ms, max_virtual_mult, {99, 100}, {1000, 999}},
         {EOS_PERCENT(chain_config.max_block_net_usage, chain_config.target_block_net_usage_pct), chain_config.max_block_net_usage, config::block_size_average_window_ms / config::block_interval_ms, max_virtual_mult, {99, 100}, {1000, 999}}
      );
      resource_limits.process_block_usage(pending->_pending_block_state->block_num);
```

之后计算hash, 首先是所有action的

```cpp
   void set_action_merkle() {
      vector<digest_type> action_digests;
      action_digests.reserve( pending->_actions.size() );
      for( const auto& a : pending->_actions )
         action_digests.emplace_back( a.digest() );

      // 计算merkle hash值
      pending->_pending_block_state->header.action_mroot = merkle( move(action_digests) );
   }
```

之后是所有交易的:

```cpp
   void set_trx_merkle() {
      vector<digest_type> trx_digests;
      const auto& trxs = pending->_pending_block_state->block->transactions;
      trx_digests.reserve( trxs.size() );
      for( const auto& a : trxs )
         trx_digests.emplace_back( a.digest() );

      pending->_pending_block_state->header.transaction_mroot = merkle( move(trx_digests) );
   }
```

最后生产区块摘要信息:

```cpp
   void create_block_summary(const block_id_type& id) {
      auto block_num = block_header::num_from_id(id);
      auto sid = block_num & 0xffff;
      db.modify( db.get<block_summary_object,by_id>(sid), [&](block_summary_object& bso ) {
          bso.block_id = id;
      });
   }

```

至此所有区块的数据已经完备.下一步是为区块签名:

```cpp
   void sign_block( const std::function<signature_type( const digest_type& )>& signer_callback  ) {
      auto p = pending->_pending_block_state;

      p->sign( signer_callback );

      static_cast<signed_block_header&>(*p->block) = p->header;
   } /// sign_block
```

签名成功之后就是调用`commit_block`

```cpp
   void commit_block( bool add_to_fork_db ) {

      // 这一步是最后一步了,执行完成之后会清理待发送的区块信息
      auto reset_pending_on_exit = fc::make_scoped_exit([this]{
         pending.reset();
      });

      try {
         // 如果是自己出的块,会默认接受
         if (add_to_fork_db) {
            pending->_pending_block_state->validated = true;
            auto new_bsp = fork_db.add(pending->_pending_block_state);
            emit(self.accepted_block_header, pending->_pending_block_state);
            head = fork_db.head();
            EOS_ASSERT(new_bsp == head, fork_database_exception, "committed block did not become the new head in fork database");
         }

         // 如果是回放 则直接写入reversible_blocks中
         if( !replaying ) {
            reversible_blocks.create<reversible_block_object>( [&]( auto& ubo ) {
               ubo.blocknum = pending->_pending_block_state->block_num;
               ubo.set_block( pending->_pending_block_state->block );
            });
         }

         // 触发accepted_block信号, 调用其它模块的回调
         emit( self.accepted_block, pending->_pending_block_state );
      } catch (...) {
         // dont bother resetting pending, instead abort the block
         reset_pending_on_exit.cancel();
         abort_block();
         throw;
      }

      // push the state for pending.
      pending->push();
   }
```

至此以上就是整个出块流程, accepted_block信号会触发net_plugin_impl中的回调,最终会调用net_plugin中的`void dispatch_manager::bcast_block (const signed_block &bsum)`函数,将新出的块广播给所有对端.

## 5.需要留意的问题