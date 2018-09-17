# EOS源码备忘-Push Transaction机制

by fanyang

---------------------------------------

这里我们讨论EOS Push Transaction 的逻辑，这块EOS与Eosforce实现有一些区别，我们会着重点出。
关于wasm相关的内容我们会有一片专门的文档分析。

我们这里通常将Transaction译做`交易`，其实这里应该是事务的意思。

## 1. Transaction与Action

在EOS中Transaction与Action是最重要的几个类型，
在EOS中，所有的链上行为都是Action，Transaction是一系列Action组成的事务。

EOS中使用继承体系划分trx与action结构，关系图如下：

```class
transaction_header <- transaction <- signed_transaction <- deferred_transaction
                        |
                 packed_transaction
```

我们这里先看一下Action的声明：

```cpp
   // 权限结构
   struct permission_level {
      account_name    actor;
      permission_name permission;
   };

   ...

   struct action {
      account_name               account;
      action_name                name;
      // 执行所需的权限
      vector<permission_level>   authorization;
      bytes                      data;

     ...

      // 打包成二进制
      template<typename T>
      T data_as()const {
         ...
      }
   };
```

Action没有什么特别的内容，但要注意：

!> 在EOS中一个transaction中包含很多个action，而在Eosforce中一个trx只能包括一个action。

下面我们分析一下transaction，这里简写为trx。

首先看下

```cpp
   /**
    *  The transaction header contains the fixed-sized data
    *  associated with each transaction. It is separated from
    *  the transaction body to facilitate partial parsing of
    *  transactions without requiring dynamic memory allocation.
    *
    *  All transactions have an expiration time after which they
    *  may no longer be included in the blockchain. Once a block
    *  with a block_header::timestamp greater than expiration is
    *  deemed irreversible, then a user can safely trust the transaction
    *  will never be included.
    *

    *  Each region is an independent blockchain, it is included as routing
    *  information for inter-blockchain communication. A contract in this
    *  region might generate or authorize a transaction intended for a foreign
    *  region.
    */
   struct transaction_header {
      time_point_sec         expiration;   ///< trx超时时间
      uint16_t               ref_block_num       = 0U; // 包含trx的block num 注意这个值是后2^16个块中
      uint32_t               ref_block_prefix    = 0UL; // blockid的低32位
      fc::unsigned_int       max_net_usage_words = 0UL; // 网络资源上限
      uint8_t                max_cpu_usage_ms    = 0; // cpu资源上限
      fc::unsigned_int       delay_sec           = 0UL; /// 延迟交易的延迟时间

      /**
       * @return the absolute block number given the relative ref_block_num
       * 计算ref_block_num
       */
      block_num_type get_ref_blocknum( block_num_type head_blocknum )const {
         return ((head_blocknum/0xffff)*0xffff) + head_blocknum%0xffff;
      }
      void set_reference_block( const block_id_type& reference_block );
      bool verify_reference_block( const block_id_type& reference_block )const;
      void validate()const;
   };

```

transaction_header包含一个trx中固定长度的数据，这里之所以要单独提出来主要是为了优化。

transaction视为交易体数据，这里主要是存储这个trx包含的action。

```cpp
   /**
    *  A transaction consits of a set of messages which must all be applied or
    *  all are rejected. These messages have access to data within the given
    *  read and write scopes.
    */
   // 在EOS中一个交易中 action要么全部执行，要么都不执行
   struct transaction : public transaction_header {
      vector<action>         context_free_actions;
      vector<action>         actions;
      extensions_type        transaction_extensions;

      // 获取trx id
      transaction_id_type        id()const;
      digest_type                sig_digest( const chain_id_type& chain_id, const vector<bytes>& cfd = vector<bytes>() )const;

     ...

   };
```

注意这里的context_free_actions，这里指上下文无关的Action，具体信息可以参见这里： https://medium.com/@bytemaster/eosio-development-update-272198df22c1 和 https://github.com/EOSIO/eos/issues/1387。
如果一个Action执行时只依赖与transaction的数据，而不依赖与链上的状态，这样的action可以并发的执行。

另外一个值得注意的是trx id：

```cpp
transaction_id_type transaction::id() const {
   digest_type::encoder enc;
   fc::raw::pack( enc, *this );
   return enc.result();
}
```

TODO eosforce的trx

这里计算trx id时完全使用trx的数据，这意味着，如果是两个trx数据完全一致，特别的他们在一个区块中，那么这两个trx的id就会是一样的。

一个trx签名之后会得到一个`signed_transaction`，

```cpp
   struct signed_transaction : public transaction
   {
      ...

      vector<signature_type>    signatures; // 签名
      vector<bytes>             context_free_data; // 上下文无关的action所使用的数据

      // 签名
      const signature_type&     sign(const private_key_type& key, const chain_id_type& chain_id);
      signature_type            sign(const private_key_type& key, const chain_id_type& chain_id)const;
      flat_set<public_key_type> get_signature_keys( const chain_id_type& chain_id, bool allow_duplicate_keys = false, bool use_cache = true )const;
   };
```

signed_transaction包含签名数据和上下文无关的action所使用的数据，

这里要谈一下`context_free_data`，可以参见 https://github.com/EOSIO/eos/commit/a41b4d56b5cbfd0346de34b0e03819f72e834041 ，之前我们看过`context_free_actions`，
在上下文无关的action中可以去从`context_free_data`获取数据，可以参见在`api_tests.cpp`中的测试用例：

```cpp

...

      {
         // back to normal action
         action act1(pl, da);
         signed_transaction trx;
         trx.context_free_actions.push_back(act);
         trx.context_free_data.emplace_back(fc::raw::pack<uint32_t>(100)); // verify payload matches context free data
         trx.context_free_data.emplace_back(fc::raw::pack<uint32_t>(200));

         trx.actions.push_back(act1);
         // attempt to access non context free api
         for (uint32_t i = 200; i <= 211; ++i) {
            trx.context_free_actions.clear();
            trx.context_free_data.clear();
            cfa.payload = i;
            cfa.cfd_idx = 1;
            action cfa_act({}, cfa);
            trx.context_free_actions.emplace_back(cfa_act);
            trx.signatures.clear();
            set_transaction_headers(trx);
            sigs = trx.sign(get_private_key(N(testapi), "active"), control->get_chain_id());
            BOOST_CHECK_EXCEPTION(push_transaction(trx), unaccessible_api,
                 [](const fc::exception& e) {
                    return expect_assert_message(e, "only context free api's can be used in this context" );
                 }
            );
         }

...

```

这里可以作为`context_free_action`的一个例子，在test_api.cpp中的合约会调用`void test_action::test_cf_action()`函数：

```cpp
// 这个是测试`context_free_action`的action
void test_action::test_cf_action() {

   eosio::action act = eosio::get_action( 0, 0 );
   cf_action cfa = act.data_as<cf_action>();
   if ( cfa.payload == 100 ) {
      // verify read of get_context_free_data, also verifies system api access
      // 测试在合约中通过 get_context_free_data 获取 context_free_data
      int size = get_context_free_data( cfa.cfd_idx, nullptr, 0 );
      eosio_assert( size > 0, "size determination failed" );
      eosio::bytes cfd( static_cast<size_t>(size) );
      size = get_context_free_data( cfa.cfd_idx, &cfd[0], static_cast<size_t>(size) );
      eosio_assert(static_cast<size_t>(size) == cfd.size(), "get_context_free_data failed" );
      uint32_t v = eosio::unpack<uint32_t>( &cfd[0], cfd.size() );
      eosio_assert( v == cfa.payload, "invalid value" );

      // 以下是测试一些功能
      // verify crypto api access
      checksum256 hash;
      char test[] = "test";

      ...

      // verify context_free_system_api
      eosio_assert( true, "verify eosio_assert can be called" );

      // 下面是测试一些在上下文无关action中不能使用的功能

   } else if ( cfa.payload == 200 ) {
      // attempt to access non context free api, privileged_api
      is_privileged(act.name);
      eosio_assert( false, "privileged_api should not be allowed" );
   } else if ( cfa.payload == 201 ) {
      // attempt to access non context free api, producer_api
      get_active_producers( nullptr, 0 );
      eosio_assert( false, "producer_api should not be allowed" );
   
    ...
   
   } else if ( cfa.payload == 211 ) {
      send_deferred( N(testapi), N(testapi), "hello", 6 );
      eosio_assert( false, "transaction_api should not be allowed" );
   }

}
```

接下来我们来看一看`packed_transaction`，通过这个类我们可以将trx打包，这样可以最大的节省空间，关于它的功能，会在下面使用的提到。

## 2. Transaction的接收和转发流程

了解Transaction类定义之后，我们先来看一下trx在EOS系统中的接收和转发流程，确定发起trx的入口，
在EOS中，大部分trx都是由用户所操纵的客户端发向同步节点，再通过同步网络发送给超级节点，超级节点会把trx打包进块，这里我们梳理一下这里的逻辑，

首先，关于客户端提交trx的流程，可以参见 https://eosforce.github.io/Documentation/#/zh-cn/eosforce_client_develop_guild ，
我们这里从node的角度看是怎么处理收到的trx的。

对于一个节点，trx可能是其他节点同步过来的，也可能是客户端通过api请求的，我们先看看api：

EOS中通过http_plugin插件响应http请求，这里我们只看处理逻辑，在`chain_api_plugin.cpp`中注册的这两个：

```cpp
void chain_api_plugin::plugin_startup() {
   ilog( "starting chain_api_plugin" );
   my.reset(new chain_api_plugin_impl(app().get_plugin<chain_plugin>().chain()));
   auto ro_api = app().get_plugin<chain_plugin>().get_read_only_api();
   auto rw_api = app().get_plugin<chain_plugin>().get_read_write_api();

   app().get_plugin<http_plugin>().add_api({

      ...

      CHAIN_RW_CALL_ASYNC(push_transaction, chain_apis::read_write::push_transaction_results, 202),
      CHAIN_RW_CALL_ASYNC(push_transactions, chain_apis::read_write::push_transactions_results, 202)
   });
}
```

最终实际调用的是这里：

```cpp
// 调用流程 push_transactions -> push_recurse -> push_transaction
void read_write::push_transaction(const read_write::push_transaction_params& params, next_function<read_write::push_transaction_results> next) {

   try {
      auto pretty_input = std::make_shared<packed_transaction>();
      auto resolver = make_resolver(this, abi_serializer_max_time);
      try {
         // 这里在使用 packed_transaction 解包
         abi_serializer::from_variant(params, *pretty_input, resolver, abi_serializer_max_time);
      } EOS_RETHROW_EXCEPTIONS(chain::packed_transaction_type_exception, "Invalid packed transaction")

      // 这里调用 incoming::methods::transaction_async 函数
      app().get_method<incoming::methods::transaction_async>()(pretty_input, true, [this, next](const fc::static_variant<fc::exception_ptr, transaction_trace_ptr>& result) -> void{
         ... // 返回返回值， 略去
      });

   } catch ( boost::interprocess::bad_alloc& ) {
      raise(SIGUSR1);
   } CATCH_AND_CALL(next);
}
```

注意这里的 persist_until_expired 参数，我们在 EOS源码备忘-Block Produce机制 这篇文档中分析过。
incoming::methods::transaction_async注册的是on_incoming_transaction_async函数：

```cpp
   my->_incoming_transaction_async_provider = app().get_method<incoming::methods::transaction_async>().register_provider([this](const packed_transaction_ptr& trx, bool persist_until_expired, next_function<transaction_trace_ptr> next) -> void {
      return my->on_incoming_transaction_async(trx, persist_until_expired, next );
   });
```

`on_incoming_transaction_async`如下：

```cpp
      void on_incoming_transaction_async(const packed_transaction_ptr& trx, bool persist_until_expired, next_function<transaction_trace_ptr> next) {
         chain::controller& chain = app().get_plugin<chain_plugin>().chain();
         if (!chain.pending_block_state()) {
            _pending_incoming_transactions.emplace_back(trx, persist_until_expired, next);
            return;
         }

         auto block_time = chain.pending_block_state()->header.timestamp.to_time_point();

         // 返回结果的回调
         auto send_response = [this, &trx, &next](const fc::static_variant<fc::exception_ptr, transaction_trace_ptr>& response) {
            next(response);
            if (response.contains<fc::exception_ptr>()) {
               _transaction_ack_channel.publish(std::pair<fc::exception_ptr, packed_transaction_ptr>(response.get<fc::exception_ptr>(), trx));
            } else {
               _transaction_ack_channel.publish(std::pair<fc::exception_ptr, packed_transaction_ptr>(nullptr, trx));
            }
         };

         auto id = trx->id();
         // 超时时间检查
         if( fc::time_point(trx->expiration()) < block_time ) {
            send_response(std::static_pointer_cast<fc::exception>(std::make_shared<expired_tx_exception>(FC_LOG_MESSAGE(error, "expired transaction ${id}", ("id", id)) )));
            return;
         }

         // 检查是否是已处理过的trx
         if( chain.is_known_unexpired_transaction(id) ) {
            send_response(std::static_pointer_cast<fc::exception>(std::make_shared<tx_duplicate>(FC_LOG_MESSAGE(error, "duplicate transaction ${id}", ("id", id)) )));
            return;
         }

         // 看看是否超过最大的执行时间了
         auto deadline = fc::time_point::now() + fc::milliseconds(_max_transaction_time_ms);
         bool deadline_is_subjective = false;
         if (_max_transaction_time_ms < 0 || (_pending_block_mode == pending_block_mode::producing && block_time < deadline) ) {
            deadline_is_subjective = true;
            deadline = block_time;
         }

         try {
            // 这里直接调用`push_transaction`来执行trx
            auto trace = chain.push_transaction(std::make_shared<transaction_metadata>(*trx), deadline);
            if (trace->except) {
               if (failure_is_subjective(*trace->except, deadline_is_subjective)) {
                  _pending_incoming_transactions.emplace_back(trx, persist_until_expired, next);
               } else {
                  auto e_ptr = trace->except->dynamic_copy_exception();
                  send_response(e_ptr);
               }
            } else {
               if (persist_until_expired) {
                  // if this trx didnt fail/soft-fail and the persist flag is set, store its ID so that we can
                  // ensure its applied to all future speculative blocks as well.
                  _persistent_transactions.insert(transaction_id_with_expiry{trx->id(), trx->expiration()});
               }
               send_response(trace);
            }

         } catch ( const guard_exception& e ) {
            app().get_plugin<chain_plugin>().handle_guard_exception(e);
         } catch ( boost::interprocess::bad_alloc& ) {
            raise(SIGUSR1);
         } CATCH_AND_CALL(send_response);
      }
```

注意上面的`is_known_unexpired_transaction`，代码如下：

```cpp
bool controller::is_known_unexpired_transaction( const transaction_id_type& id) const {
   return db().find<transaction_object, by_trx_id>(id);
}
```

与之对应的是这个函数：

```cpp
   void transaction_context::record_transaction( const transaction_id_type& id, fc::time_point_sec expire ) {
      try {
          control.db().create<transaction_object>([&](transaction_object& transaction) {
              transaction.trx_id = id;
              transaction.expiration = expire;
          });
      } catch( const boost::interprocess::bad_alloc& ) {
         throw;
      } catch ( ... ) {
          EOS_ASSERT( false, tx_duplicate,
                     "duplicate transaction ${id}", ("id", id ) );
      }
   } /// record_transaction
```

在`push_transaction`中会调用到，记录trx已经被处理过了。

下面我们来看看`send_response`这个回调：

```cpp
         auto send_response = [this, &trx, &next](const fc::static_variant<fc::exception_ptr, transaction_trace_ptr>& response) {
            next(response);
            if (response.contains<fc::exception_ptr>()) {
               _transaction_ack_channel.publish(std::pair<fc::exception_ptr, packed_transaction_ptr>(response.get<fc::exception_ptr>(), trx));
            } else {
               _transaction_ack_channel.publish(std::pair<fc::exception_ptr, packed_transaction_ptr>(nullptr, trx));
            }
         };
```

在执行之后会调用`send_response`，这里是将结果发送到`_transaction_ack_channel`中，对于`_transaction_ack_channel`，
这个实际对应的是下面这个类型：

```cpp
   namespace compat {
      namespace channels {
         using transaction_ack       = 
            channel_decl<struct accepted_transaction_tag, std::pair<fc::exception_ptr, packed_transaction_ptr>>;
      }
   }
```

在EOS中在net_plugin注册响应这个channel的函数:

```cpp
      my->incoming_transaction_ack_subscription = 
            app().get_channel<channels::transaction_ack>().subscribe(
                  boost::bind(&net_plugin_impl::transaction_ack, my.get(), _1));
```

处理的函数如下：

```cpp
   void net_plugin_impl::transaction_ack(const std::pair<fc::exception_ptr, packed_transaction_ptr>& results) {
      transaction_id_type id = results.second->id();
      if (results.first) {
         fc_ilog(logger,"signaled NACK, trx-id = ${id} : ${why}",("id", id)("why", results.first->to_detail_string()));
         dispatcher->rejected_transaction(id);
      } else {
         fc_ilog(logger,"signaled ACK, trx-id = ${id}",("id", id));
         dispatcher->bcast_transaction(*results.second);
      }
   }
```

这里会将运行正常的广播给其他节点，这其中会发送给超级节点打包如块。

## 3. `push_transaction`代码分析

这里我们来分析下`push_transaction`的过程，作为执行trx的入口，这个函数在EOS中非常重要，另一方面，这里EOS与Eosforce有一定区别，这里会具体介绍。

我们先来看下`push_transaction`的`transaction_metadata`参数， 这个参数统一了各种不同类型，不同行为的trx：

```cpp
/**
 *  This data structure should store context-free cached data about a transaction such as
 *  packed/unpacked/compressed and recovered keys
 */
class transaction_metadata {
   public:
      transaction_id_type                                        id;  // trx ID
      transaction_id_type                                        signed_id; // signed trx ID
      signed_transaction                                         trx;
      packed_transaction                                         packed_trx;
      optional<pair<chain_id_type, flat_set<public_key_type>>>   signing_keys;
      bool                                                       accepted = false; // 标注是否调用了accepted信号，确保只调用一次
      bool                                                       implicit = false; // 是否忽略检查
      bool                                                       scheduled = false; // 是否是延迟trx

      explicit transaction_metadata( const signed_transaction& t, packed_transaction::compression_type c = packed_transaction::none )
      :trx(t),packed_trx(t, c) {
         id = trx.id();
         //raw_packed = fc::raw::pack( static_cast<const transaction&>(trx) );
         signed_id = digest_type::hash(packed_trx);
      }

      explicit transaction_metadata( const packed_transaction& ptrx )
      :trx( ptrx.get_signed_transaction() ), packed_trx(ptrx) {
         id = trx.id();
         //raw_packed = fc::raw::pack( static_cast<const transaction&>(trx) );
         signed_id = digest_type::hash(packed_trx);
      }

      const flat_set<public_key_type>& recover_keys( const chain_id_type& chain_id );

      uint32_t total_actions()const { return trx.context_free_actions.size() + trx.actions.size(); }
};

using transaction_metadata_ptr = std::shared_ptr<transaction_metadata>;
```

先看一下`implicit`，这个参数指示下面的逻辑是否要忽略对于trx的各种检查，一般用于系统内部的trx，
对于EOS，主要是处理`on_block_transaction`（可以参见出块文档），在`start_block`调用：

```cpp

...

            auto onbtrx = std::make_shared<transaction_metadata>( get_on_block_transaction() );
            onbtrx->implicit = true; // on_block trx 会被无条件接受
            auto reset_in_trx_requiring_checks = fc::make_scoped_exit([old_value=in_trx_requiring_checks,this](){
                  in_trx_requiring_checks = old_value;
               });
            in_trx_requiring_checks = true; // 修改in_trx_requiring_checks变量达到不将trx写入区块，一些系统的trx没有必要写入区块。
            push_transaction( onbtrx, fc::time_point::maximum(), self.get_global_properties().configuration.min_transaction_cpu_usage, true );

...

```

!> Eosforce不同之处

而对于EOSForce中，除了on_block action之外，onfee合约也是被设置为implicit==true的，onfee合约是eosforce的系统合约，设计用来收取交易的手续费。

下面我们逐行分析下代码，EOS中`push_transaction`代码如下：

```cpp
   /**
    *  This is the entry point for new transactions to the block state. It will check authorization and
    *  determine whether to execute it now or to delay it. Lastly it inserts a transaction receipt into
    *  the pending block.
    */
   transaction_trace_ptr push_transaction( const transaction_metadata_ptr& trx,
                                           fc::time_point deadline,
                                           uint32_t billed_cpu_time_us,
                                           bool explicit_billed_cpu_time = false )
   {
      // deadline必须不为空
      // deadline是trx执行时间的一个大上限，为了防止某些trx运行时间过长导致出块失败等问题，
      // 这里必须有一个严格的上限，一旦超过上限，交易会立即失败。
      EOS_ASSERT(deadline != fc::time_point(), transaction_exception, "deadline cannot be uninitialized");

      transaction_trace_ptr trace; // trace主要用来保存执行中的一些错误信息。
      try {
         // trx_context是执行trx的上下文状态，下面会专门说明
         transaction_context trx_context(self, trx->trx, trx->id);
         if ((bool)subjective_cpu_leeway && pending->_block_status == controller::block_status::incomplete) {
            trx_context.leeway = *subjective_cpu_leeway;
         }

         // 设置数据
         trx_context.deadline = deadline;
         trx_context.explicit_billed_cpu_time = explicit_billed_cpu_time;
         trx_context.billed_cpu_time_us = billed_cpu_time_us;
         trace = trx_context.trace;
         try {
            if( trx->implicit ) { // 如果是implicit的就没有必要做下面的一些检查和记录，这里的检查主要是资源方面的
               trx_context.init_for_implicit_trx();
               trx_context.can_subjectively_fail = false;
            } else {
               // 如果是重放并且不是重放过程中接到的新交易，则不去使用`record_transaction`记录
               bool skip_recording = replay_head_time && (time_point(trx->trx.expiration) <= *replay_head_time);

               // 一些trx_context的初始化操作
               trx_context.init_for_input_trx( trx->packed_trx.get_unprunable_size(),
                                               trx->packed_trx.get_prunable_size(),
                                               trx->trx.signatures.size(),
                                               skip_recording);
            }

            if( trx_context.can_subjectively_fail && pending->_block_status == controller::block_status::incomplete ) {
               check_actor_list( trx_context.bill_to_accounts ); // Assumes bill_to_accounts is the set of actors authorizing the transaction
            }


            trx_context.delay = fc::seconds(trx->trx.delay_sec);

            if( !self.skip_auth_check() && !trx->implicit ) {
               // 检测交易所需要的权限
               authorization.check_authorization(
                       trx->trx.actions,
                       trx->recover_keys( chain_id ),
                       {},
                       trx_context.delay,
                       [](){}
                       /*std::bind(&transaction_context::add_cpu_usage_and_check_time, &trx_context,
                                 std::placeholders::_1)*/,
                       false
               );
            }

            // 执行，注意这时trx_context包括所有信息和状态
            trx_context.exec();
            trx_context.finalize(); // Automatically rounds up network and CPU usage in trace and bills payers if successful

            auto restore = make_block_restore_point();

            if (!trx->implicit) {
               // 如果是非implicit的交易，则需要进入区块。
               transaction_receipt::status_enum s = (trx_context.delay == fc::seconds(0))
                                                    ? transaction_receipt::executed
                                                    : transaction_receipt::delayed;
               trace->receipt = push_receipt(trx->packed_trx, s, trx_context.billed_cpu_time_us, trace->net_usage);
               pending->_pending_block_state->trxs.emplace_back(trx);
            } else {
               // 注意，这里implicit类的交易是不会进入区块的，只会计入资源消耗
               // 因为这类的trx无条件运行，所以不需要另行记录。
               transaction_receipt_header r;
               r.status = transaction_receipt::executed;
               r.cpu_usage_us = trx_context.billed_cpu_time_us;
               r.net_usage_words = trace->net_usage / 8;
               trace->receipt = r;
            }

            // 这里会将执行过的action写入待出块状态的_actions之中
            fc::move_append(pending->_actions, move(trx_context.executed));

            // call the accept signal but only once for this transaction
            // 为这个交易调用accept信号，保证只调用一次
            if (!trx->accepted) {
               trx->accepted = true;
               emit( self.accepted_transaction, trx);
            }

            // 触发applied_transaction信号
            emit(self.applied_transaction, trace);


            if ( read_mode != db_read_mode::SPECULATIVE && pending->_block_status == controller::block_status::incomplete ) {
               //this may happen automatically in destructor, but I prefere make it more explicit
               trx_context.undo();
            } else {
               restore.cancel();
               trx_context.squash();
            }

            // implicit的trx压根没有在unapplied_transactions中
            if (!trx->implicit) {
               unapplied_transactions.erase( trx->signed_id );
            }
            return trace;
         } catch (const fc::exception& e) {
            trace->except = e;
            trace->except_ptr = std::current_exception();
         }

         // 注意这里，如果成功的话上面就返回了这里是失败的情况
         // failure_is_subjective 表明
         if (!failure_is_subjective(*trace->except)) {
            unapplied_transactions.erase( trx->signed_id );
         }

         emit( self.accepted_transaction, trx );
         emit( self.applied_transaction, trace );

         return trace;
      } FC_CAPTURE_AND_RETHROW((trace))
   } /// push_transaction
```

上面注释中阐述了大致的流程，下面仔细分析一下：

首先是`trx_context`，这个对象的类声明如下：

```cpp
   class transaction_context {
         ... // 省略

         void dispatch_action( action_trace& trace, const action& a, account_name receiver, bool context_free = false, uint32_t recurse_depth = 0 );
         inline void dispatch_action( action_trace& trace, const action& a, bool context_free = false ) {
            dispatch_action(trace, a, a.account, context_free);
         };
         void schedule_transaction();
         void record_transaction( const transaction_id_type& id, fc::time_point_sec expire );

         void validate_cpu_usage_to_bill( int64_t u, bool check_minimum = true )const;

      public:
         controller&                   control; // controller类的引用
         const signed_transaction&     trx; // 要执行的trx
         transaction_id_type           id;
         optional<chainbase::database::session>  undo_session;
         transaction_trace_ptr         trace; // 记录错误的trace
         fc::time_point                start; // 起始时刻

         fc::time_point                published; // publish的时刻


         vector<action_receipt>        executed; // 执行完成的action
         flat_set<account_name>        bill_to_accounts; 
         flat_set<account_name>        validate_ram_usage;

         /// the maximum number of virtual CPU instructions of the transaction that can be safely billed to the billable accounts
         uint64_t                      initial_max_billable_cpu = 0;

         fc::microseconds              delay;
         bool                          is_input           = false;
         bool                          apply_context_free = true;
         bool                          can_subjectively_fail = true;

         fc::time_point                deadline = fc::time_point::maximum();
         fc::microseconds              leeway = fc::microseconds(3000);
         int64_t                       billed_cpu_time_us = 0;
         bool                          explicit_billed_cpu_time = false;

      private:
         bool                          is_initialized = false;


         uint64_t                      net_limit = 0;
         bool                          net_limit_due_to_block = true;
         bool                          net_limit_due_to_greylist = false;
         uint64_t                      eager_net_limit = 0;
         uint64_t&                     net_usage; /// reference to trace->net_usage

         bool                          cpu_limit_due_to_greylist = false;

         fc::microseconds              initial_objective_duration_limit;
         fc::microseconds              objective_duration_limit;
         fc::time_point                _deadline = fc::time_point::maximum();
         int64_t                       deadline_exception_code = block_cpu_usage_exceeded::code_value;
         int64_t                       billing_timer_exception_code = block_cpu_usage_exceeded::code_value;
         fc::time_point                pseudo_start;
         fc::microseconds              billed_time;
         fc::microseconds              billing_timer_duration_limit;
   };
```

我们先看一下`init_for_input_trx`：

```cpp
   void transaction_context::init_for_input_trx( uint64_t packed_trx_unprunable_size, // 这个是指trx打包后完整的大小
                                                 uint64_t packed_trx_prunable_size, // 这个指trx额外信息的大小
                                                 uint32_t num_signatures, // 这个参数没用上
                                                 bool skip_recording ) // 是否要跳过记录
   {
      // 根据cfg和trx初始化资源
      const auto& cfg = control.get_global_properties().configuration;

      // 利用packed_trx_unprunable_size和packed_trx_prunable_size 计算net资源消耗
      uint64_t discounted_size_for_pruned_data = packed_trx_prunable_size;
      if( cfg.context_free_discount_net_usage_den > 0
          && cfg.context_free_discount_net_usage_num < cfg.context_free_discount_net_usage_den )
      {
         discounted_size_for_pruned_data *= cfg.context_free_discount_net_usage_num;
         discounted_size_for_pruned_data =  ( discounted_size_for_pruned_data + cfg.context_free_discount_net_usage_den - 1)
                                                                                    / cfg.context_free_discount_net_usage_den; // rounds up
      }

      uint64_t initial_net_usage = static_cast<uint64_t>(cfg.base_per_transaction_net_usage)
                                    + packed_trx_unprunable_size + discounted_size_for_pruned_data;


      // 对于delay trx需要额外的net资源
      if( trx.delay_sec.value > 0 ) {
          // If delayed, also charge ahead of time for the additional net usage needed to retire the delayed transaction
          // whether that be by successfully executing, soft failure, hard failure, or expiration.
         initial_net_usage += static_cast<uint64_t>(cfg.base_per_transaction_net_usage)
                               + static_cast<uint64_t>(config::transaction_id_net_usage);
      }

      // 初始化一些信息
      published = control.pending_block_time();
      is_input = true;
      if (!control.skip_trx_checks()) {
         control.validate_expiration(trx);
         control.validate_tapos(trx);
         control.validate_referenced_accounts(trx);
      }
      init( initial_net_usage); // 这里调用init函数， 在这个函数中会处理cpu资源和ram资源
      if (!skip_recording)
         // 将trx添加入记录中
         record_transaction( id, trx.expiration ); /// checks for dupes
   }
```

这里会先计算net，再在init函数中处理其他资源：

```cpp
   void transaction_context::init(uint64_t initial_net_usage)
   {
      EOS_ASSERT( !is_initialized, transaction_exception, "cannot initialize twice" );
      const static int64_t large_number_no_overflow = std::numeric_limits<int64_t>::max()/2;

      const auto& cfg = control.get_global_properties().configuration;
      auto& rl = control.get_mutable_resource_limits_manager();

      net_limit = rl.get_block_net_limit();

      objective_duration_limit = fc::microseconds( rl.get_block_cpu_limit() );
      _deadline = start + objective_duration_limit;

      // Possibly lower net_limit to the maximum net usage a transaction is allowed to be billed
      if( cfg.max_transaction_net_usage <= net_limit ) {
         net_limit = cfg.max_transaction_net_usage;
         net_limit_due_to_block = false;
      }

      // Possibly lower objective_duration_limit to the maximum cpu usage a transaction is allowed to be billed
      if( cfg.max_transaction_cpu_usage <= objective_duration_limit.count() ) {
         objective_duration_limit = fc::microseconds(cfg.max_transaction_cpu_usage);
         billing_timer_exception_code = tx_cpu_usage_exceeded::code_value;
         _deadline = start + objective_duration_limit;
      }

      // Possibly lower net_limit to optional limit set in the transaction header
      uint64_t trx_specified_net_usage_limit = static_cast<uint64_t>(trx.max_net_usage_words.value) * 8;
      if( trx_specified_net_usage_limit > 0 && trx_specified_net_usage_limit <= net_limit ) {
         net_limit = trx_specified_net_usage_limit;
         net_limit_due_to_block = false;
      }

      // Possibly lower objective_duration_limit to optional limit set in transaction header
      if( trx.max_cpu_usage_ms > 0 ) {
         auto trx_specified_cpu_usage_limit = fc::milliseconds(trx.max_cpu_usage_ms);
         if( trx_specified_cpu_usage_limit <= objective_duration_limit ) {
            objective_duration_limit = trx_specified_cpu_usage_limit;
            billing_timer_exception_code = tx_cpu_usage_exceeded::code_value;
            _deadline = start + objective_duration_limit;
         }
      }

      initial_objective_duration_limit = objective_duration_limit;

      if( billed_cpu_time_us > 0 ) // could also call on explicit_billed_cpu_time but it would be redundant
         validate_cpu_usage_to_bill( billed_cpu_time_us, false ); // Fail early if the amount to be billed is too high

      // Record accounts to be billed for network and CPU usage
      for( const auto& act : trx.actions ) {
         for( const auto& auth : act.authorization ) {
            bill_to_accounts.insert( auth.actor );
         }
      }
      validate_ram_usage.reserve( bill_to_accounts.size() );

      // Update usage values of accounts to reflect new time
      rl.update_account_usage( bill_to_accounts, block_timestamp_type(control.pending_block_time()).slot );

      // Calculate the highest network usage and CPU time that all of the billed accounts can afford to be billed
      int64_t account_net_limit = 0;
      int64_t account_cpu_limit = 0;
      bool greylisted_net = false, greylisted_cpu = false;
      std::tie( account_net_limit, account_cpu_limit, greylisted_net, greylisted_cpu) = max_bandwidth_billed_accounts_can_pay();
      net_limit_due_to_greylist |= greylisted_net;
      cpu_limit_due_to_greylist |= greylisted_cpu;

      eager_net_limit = net_limit;

      // Possible lower eager_net_limit to what the billed accounts can pay plus some (objective) leeway
      auto new_eager_net_limit = std::min( eager_net_limit, static_cast<uint64_t>(account_net_limit + cfg.net_usage_leeway) );
      if( new_eager_net_limit < eager_net_limit ) {
         eager_net_limit = new_eager_net_limit;
         net_limit_due_to_block = false;
      }

      // Possibly limit deadline if the duration accounts can be billed for (+ a subjective leeway) does not exceed current delta
      if( (fc::microseconds(account_cpu_limit) + leeway) <= (_deadline - start) ) {
         _deadline = start + fc::microseconds(account_cpu_limit) + leeway;
         billing_timer_exception_code = leeway_deadline_exception::code_value;
      }

      billing_timer_duration_limit = _deadline - start;

      // Check if deadline is limited by caller-set deadline (only change deadline if billed_cpu_time_us is not set)
      if( explicit_billed_cpu_time || deadline < _deadline ) {
         _deadline = deadline;
         deadline_exception_code = deadline_exception::code_value;
      } else {
         deadline_exception_code = billing_timer_exception_code;
      }

      eager_net_limit = (eager_net_limit/8)*8; // Round down to nearest multiple of word size (8 bytes) so check_net_usage can be efficient

      if( initial_net_usage > 0 )
         add_net_usage( initial_net_usage );  // Fail early if current net usage is already greater than the calculated limit

      checktime(); // Fail early if deadline has already been exceeded

      is_initialized = true;
   }
```

以上就是`transaction_context`初始化过程，这里主要是处理资源消耗。

下面是`exec`函数，这个函数很简单：

```cpp
   void transaction_context::exec() {
      EOS_ASSERT( is_initialized, transaction_exception, "must first initialize" );

      // 调用`dispatch_action`，这里并没有对上下文无关trx进行特别的操作，只是参数不同
      if( apply_context_free ) {
         for( const auto& act : trx.context_free_actions ) {
            trace->action_traces.emplace_back();
            dispatch_action( trace->action_traces.back(), act, true );
         }
      }

      if( delay == fc::microseconds() ) {
         for( const auto& act : trx.actions ) {
            trace->action_traces.emplace_back();
            dispatch_action( trace->action_traces.back(), act );
         }
      } else {
         // 对于延迟交易，这里特别处理
         schedule_transaction();
      }
   }
```

主要执行在`dispatch_action`中，这里会根据action不同分别触发对应的调用：

```cpp
   void transaction_context::dispatch_action( action_trace& trace, const action& a, account_name receiver, bool context_free, uint32_t recurse_depth ) {
      // 构建apply_context执行action， apply_context的分析在下节进行
      apply_context  acontext( control, *this, a, recurse_depth );
      acontext.context_free = context_free;
      acontext.receiver     = receiver;

      try {
         acontext.exec();
      } catch( ... ) {
         trace = move(acontext.trace);
         throw;
      }

      // 汇总结果到trace
      trace = move(acontext.trace);
   }
```

对于延迟交易，执行`schedule_transaction`：

```cpp
   void transaction_context::schedule_transaction() {
      // 因为交易延迟执行，会消耗额外的net和ram资源
      // Charge ahead of time for the additional net usage needed to retire the delayed transaction
      // whether that be by successfully executing, soft failure, hard failure, or expiration.
      if( trx.delay_sec.value == 0 ) { // Do not double bill. Only charge if we have not already charged for the delay.
         const auto& cfg = control.get_global_properties().configuration;
         add_net_usage( static_cast<uint64_t>(cfg.base_per_transaction_net_usage)
                         + static_cast<uint64_t>(config::transaction_id_net_usage) ); // Will exit early if net usage cannot be payed.
      }

      auto first_auth = trx.first_authorizor();

      // 将延迟交易写入节点运行时状态数据库中，到时会从这里查找出来执行
      uint32_t trx_size = 0;
      const auto& cgto = control.db().create<generated_transaction_object>( [&]( auto& gto ) {
        gto.trx_id      = id;
        gto.payer       = first_auth;
        gto.sender      = account_name(); /// delayed transactions have no sender
        gto.sender_id   = transaction_id_to_sender_id( gto.trx_id );
        gto.published   = control.pending_block_time();
        gto.delay_until = gto.published + delay;
        gto.expiration  = gto.delay_until + fc::seconds(control.get_global_properties().configuration.deferred_trx_expiration_window);
        trx_size = gto.set( trx );
      });

      // 因为要写内存记录，所以也消耗了一定的ram
      add_ram_usage( cgto.payer, (config::billable_size_v<generated_transaction_object> + trx_size) );
   }
```

调用完exec之后会调用`transaction_context::finalize()`：

```cpp
   void transaction_context::finalize() {
      EOS_ASSERT( is_initialized, transaction_exception, "must first initialize" );

      if( is_input ) {
         auto& am = control.get_mutable_authorization_manager();
         for( const auto& act : trx.actions ) {
            for( const auto& auth : act.authorization ) {
               am.update_permission_usage( am.get_permission(auth) );
            }
         }
      }

      auto& rl = control.get_mutable_resource_limits_manager();
      for( auto a : validate_ram_usage ) {
         rl.verify_account_ram_usage( a );
      }

      // Calculate the new highest network usage and CPU time that all of the billed accounts can afford to be billed
      int64_t account_net_limit = 0;
      int64_t account_cpu_limit = 0;
      bool greylisted_net = false, greylisted_cpu = false;
      std::tie( account_net_limit, account_cpu_limit, greylisted_net, greylisted_cpu) = max_bandwidth_billed_accounts_can_pay();
      net_limit_due_to_greylist |= greylisted_net;
      cpu_limit_due_to_greylist |= greylisted_cpu;

      // Possibly lower net_limit to what the billed accounts can pay
      if( static_cast<uint64_t>(account_net_limit) <= net_limit ) {
         // NOTE: net_limit may possibly not be objective anymore due to net greylisting, but it should still be no greater than the truly objective net_limit
         net_limit = static_cast<uint64_t>(account_net_limit);
         net_limit_due_to_block = false;
      }

      // Possibly lower objective_duration_limit to what the billed accounts can pay
      if( account_cpu_limit <= objective_duration_limit.count() ) {
         // NOTE: objective_duration_limit may possibly not be objective anymore due to cpu greylisting, but it should still be no greater than the truly objective objective_duration_limit
         objective_duration_limit = fc::microseconds(account_cpu_limit);
         billing_timer_exception_code = tx_cpu_usage_exceeded::code_value;
      }

      net_usage = ((net_usage + 7)/8)*8; // Round up to nearest multiple of word size (8 bytes)

      eager_net_limit = net_limit;
      check_net_usage();

      auto now = fc::time_point::now();
      trace->elapsed = now - start;

      update_billed_cpu_time( now );

      validate_cpu_usage_to_bill( billed_cpu_time_us );

      rl.add_transaction_usage( bill_to_accounts, static_cast<uint64_t>(billed_cpu_time_us), net_usage,
                                block_timestamp_type(control.pending_block_time()).slot ); // Should never fail
   }
```

接下来调用`make_block_restore_point`：

```cpp
   // The returned scoped_exit should not exceed the lifetime of the pending which existed when make_block_restore_point was called.
   fc::scoped_exit<std::function<void()>> make_block_restore_point() {
      auto orig_block_transactions_size = pending->_pending_block_state->block->transactions.size();
      auto orig_state_transactions_size = pending->_pending_block_state->trxs.size();
      auto orig_state_actions_size      = pending->_actions.size();

      std::function<void()> callback = [this,
                                        orig_block_transactions_size,
                                        orig_state_transactions_size,
                                        orig_state_actions_size]()
      {
         pending->_pending_block_state->block->transactions.resize(orig_block_transactions_size);
         pending->_pending_block_state->trxs.resize(orig_state_transactions_size);
         pending->_actions.resize(orig_state_actions_size);
      };

      return fc::make_scoped_exit( std::move(callback) );
   }
```

而后对于不是implicit的交易会调用`push_receipt`，这里会将trx写入区块数据中，这也意味着implicit为true的交易虽然执行了，但不会在区块中。

```cpp
   /**
    *  Adds the transaction receipt to the pending block and returns it.
    */
   template<typename T>
   const transaction_receipt& push_receipt( const T& trx, transaction_receipt_header::status_enum status,
                                            uint64_t cpu_usage_us, uint64_t net_usage ) {
      uint64_t net_usage_words = net_usage / 8;
      EOS_ASSERT( net_usage_words*8 == net_usage, transaction_exception, "net_usage is not divisible by 8" );
      pending->_pending_block_state->block->transactions.emplace_back( trx );
      transaction_receipt& r = pending->_pending_block_state->block->transactions.back();
      r.cpu_usage_us         = cpu_usage_us;
      r.net_usage_words      = net_usage_words;
      r.status               = status;
      return r;
   }
```

上面的逻辑很大程度上和implicit为true时的逻辑重复，估计以后会重构。

接下来值得注意的是这里：

```cpp
            if ( read_mode != db_read_mode::SPECULATIVE && pending->_block_status == controller::block_status::incomplete ) {
               //this may happen automatically in destructor, but I prefere make it more explicit
               trx_context.undo();
            } else {
               restore.cancel();
               trx_context.squash();
            }
```

这里调用`database::session`对应的函数，

!> Eosforce不同之处

以上是EOS的流程，这里我们再来看看Eosforce的不同之处，Eosforce与EOS一个明显的不同是Eosforce采用了基于手续费的资源模型，
这种模型意味着，如果一个交易在超级节点打包进块时失败了，此时也要收取手续费，否则会造成潜在的攻击风险，所以Eosforce中，执行失败的交易也会写入区块中，这样每次执行时会调用对应onfee。
另一方面, Eosforce虽然使用手续费，但是还是区分cpu，net，ram资源，并且在大的限制上依然进行检查。
后续Eosforce会完成新的资源模型，这里会有所改动。

Eosforce中的`push_transaction`函数如下：

```cpp
   transaction_trace_ptr push_transaction( const transaction_metadata_ptr& trx,
                                           fc::time_point deadline,
                                           uint32_t billed_cpu_time_us,
                                           bool explicit_billed_cpu_time = false )
   {
      EOS_ASSERT(deadline != fc::time_point(), transaction_exception, "deadline cannot be uninitialized");

      // eosforce暂时没有开放延迟交易和上下文无关交易
      EOS_ASSERT(trx->trx.delay_sec.value == 0UL, transaction_exception, "delay,transaction failed");
      EOS_ASSERT(trx->trx.context_free_actions.size()==0, transaction_exception, "context free actions size should be zero!");
      
      // 在eosforce中，为了安全性，对于特定一些交易进行了额外的验证，主要是考虑到，系统会将执行错误的交易写入区块
      // 此时就要先验证下交易内容，特别是大小上有没有超出限制，否则将会带来安全问题。
      check_action(trx->trx.actions);

      transaction_trace_ptr trace;
      try {
         // 一样的代码 略去
         ...

         try {
            // 一样的代码 略去
            ...

               // 处理手续费
               EOS_ASSERT(trx->trx.fee == txfee.get_required_fee(trx->trx), transaction_exception, "set tx fee failed");
               EOS_ASSERT(txfee.check_transaction(trx->trx) == true, transaction_exception, "transaction include actor more than one");
               try {
                  // 这里会执行onfee合约，也是通过`push_transaction`实现的
                  auto onftrx = std::make_shared<transaction_metadata>( get_on_fee_transaction(trx->trx.fee, trx->trx.actions[0].authorization[0].actor) );
                  onftrx->implicit = true;
                  auto onftrace = push_transaction( onftrx, fc::time_point::maximum(), config::default_min_transaction_cpu_usage, true);
                  // 这里如果执行失败直接抛出异常，不会执行下面的东西
                  if( onftrace->except ) throw *onftrace->except;
               }  catch (const fc::exception &e) {
                  EOS_ASSERT(false, transaction_exception, "on fee transaction failed, exception: ${e}", ("e", e));
               }  catch ( ... ) {
                  EOS_ASSERT(false, transaction_exception, "on fee transaction failed, but shouldn't enough asset to pay for transaction fee");
               }
            }

            // 注意这一层try catch，因为eos中出错的交易会被抛弃，所以eos的异常会被直接抛出到外层
            // 而在eosforce中出错的交易会进入区块
            // 但是要注意，这里如果这里并不是在超级节点出块时调用，虽然也会执行下面的逻辑，但是不会被转发给超级节点。
            try {
              if(explicit_billed_cpu_time && billed_cpu_time_us == 0){
                // 在eosforce中 因为超级节点打包区块时失败的交易也会被写入区块中，
                // 而很多交易失败的原因不是交易本身有问题，而是在执行交易时，资源上限被触发，导致交易被直接判定为失败，
                // 这时写入区块的交易的cpu消耗是0, 这里是需要失败的，否则重跑区块时会出现不同步的情况
                EOS_ASSERT(false, transaction_exception, "billed_cpu_time_us is 0");
              }
               trx_context.exec();
               trx_context.finalize(); // Automatically rounds up network and CPU usage in trace and bills payers if successful
             } catch (const fc::exception &e) {
               trace->except = e;
               trace->except_ptr = std::current_exception();

               // eosforce加了一些日志
               if (head->block_num != 1) {
                 elog("---trnasction exe failed--------trace: ${trace}", ("trace", trace));
               }
             }

            auto restore = make_block_restore_point();

            if (!trx->implicit) {
               // 这里不太好的地方是，对于出错的交易也被标为`executed`（严格说也确实是executed），后续eosforce将会重构这里
               transaction_receipt::status_enum s = (trx_context.delay == fc::seconds(0))
                                                    ? transaction_receipt::executed
                                                    : transaction_receipt::delayed;
               trace->receipt = push_receipt(trx->packed_trx, s, trx_context.billed_cpu_time_us, trace->net_usage);
               pending->_pending_block_state->trxs.emplace_back(trx);
            } else {
               transaction_receipt_header r;
               r.status = transaction_receipt::executed;
               r.cpu_usage_us = trx_context.billed_cpu_time_us;
               r.net_usage_words = trace->net_usage / 8;
               trace->receipt = r;
            }

            // 以下是相同的

      } FC_CAPTURE_AND_RETHROW((trace))
   } /// push_transaction
```

可以看出主要不同就是手续费导致的，这里必须要注意，就是eosforce中区块内会包括一些出错的交易。

## 4. `apply_context`代码分析

这里我们来看看action的执行过程，上面在`dispatch_action`中创建`apply_context`执行action，我们这里分析这一块的代码。

`apply_context`结构如下：

```cpp

```

## 5. 几个内置的action

## 5. 需要留意的问题