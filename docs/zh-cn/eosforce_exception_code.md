# EOS EXCEPTION CODE    异常码


##  (TYPE, BASE, CODE, WHAT)  (类型，基类型，异常码，描述信息):

- ( chain_exception, 3000000, "blockchain exception" )

- ( chain_type_exception, chain_exception, 3010000, "chain type exception" )
- ( name_type_exception,               chain_type_exception, 3010001, "Invalid name" )
- ( public_key_type_exception,         chain_type_exception, 3010002, "Invalid public key" )
- ( private_key_type_exception,        chain_type_exception, 3010003, "Invalid private key" )
- ( authority_type_exception,          chain_type_exception, 3010004, "Invalid authority" )
- ( action_type_exception,             chain_type_exception, 3010005, "Invalid action" )
- ( transaction_type_exception,        chain_type_exception, 3010006, "Invalid transaction" )
- ( abi_type_exception,                chain_type_exception, 3010007, "Invalid ABI"  )
- ( block_id_type_exception,           chain_type_exception, 3010008, "Invalid block ID" )
- ( transaction_id_type_exception,     chain_type_exception, 3010009, "Invalid transaction ID" )
- ( packed_transaction_type_exception, chain_type_exception, 3010010, "Invalid packed transaction" )
- ( asset_type_exception,              chain_type_exception, 3010011, "Invalid asset" )
- ( chain_id_type_exception,           chain_type_exception, 3010012, "Invalid chain ID" )
- ( fixed_key_type_exception,          chain_type_exception, 3010013, "Invalid fixed key" )
- ( symbol_type_exception,             chain_type_exception, 3010014, "Invalid symbol" )

- ( fork_database_exception,            chain_exception, 3020000, "Fork database exception" )
- ( fork_db_block_not_found,            fork_database_exception, 3020001, "Block can not be found" )


- ( block_validate_exception, chain_exception, 3030000, "Block exception" )


- ( unlinkable_block_exception, block_validate_exception, 3030001, "Unlinkable block" )
- ( block_tx_output_exception,   block_validate_exception, 3030002, "Transaction outputs in block do not match transaction outputs from applying block" )
- ( block_concurrency_exception, block_validate_exception, 3030003, "Block does not guarantee concurrent execution without conflicts" )
- ( block_lock_exception,        block_validate_exception, 3030004, "Shard locks in block are incorrect or mal-formed" )
- ( block_resource_exhausted,    block_validate_exception, 3030005, "Block exhausted allowed resources" )
- ( block_too_old_exception,     block_validate_exception, 3030006, "Block is too old to push" )
- ( block_from_the_future,       block_validate_exception, 3030007, "Block is from the future" )
- ( wrong_signing_key,           block_validate_exception, 3030008, "Block is not signed with expected key" )
- ( wrong_producer,              block_validate_exception, 3030009, "Block is not signed by expected producer" )

- ( transaction_exception,             chain_exception, 3040000, "Transaction exception" )

- ( tx_decompression_error,      transaction_exception, 3040001, "Error decompressing transaction" )
- ( tx_no_action,                transaction_exception, 3040002, "Transaction should have at least one normal action" )
- ( tx_no_auths,                 transaction_exception, 3040003, "Transaction should have at least one required authority" )
- ( cfa_irrelevant_auth,         transaction_exception, 3040004, "Context-free action should have no required authority" )
- ( expired_tx_exception,        transaction_exception, 3040005, "Expired Transaction" )
- ( tx_exp_too_far_exception,    transaction_exception, 3040006, "Transaction Expiration Too Far" )
- ( invalid_ref_block_exception, transaction_exception, 3040007, "Invalid Reference Block" )
- ( tx_duplicate,                transaction_exception, 3040008, "Duplicate transaction" )
- ( deferred_tx_duplicate,       transaction_exception, 3040009, "Duplicate deferred transaction" )
- ( cfa_inside_generated_tx,     transaction_exception, 3040010, "Context free action is not allowed inside generated transaction" )
- ( tx_not_found,     transaction_exception, 3040011, "The transaction can not be found" )
- ( too_many_tx_at_once,          transaction_exception, 3040012, "Pushing too many transactions at once" )
- ( tx_too_big,                   transaction_exception, 3040013, "Transaction is too big" )
- ( unknown_transaction_compression, transaction_exception, 3040014, "Unknown transaction compression" )
- ( tx_too_much, transaction_exception, 3040100, "block txs must less than config::block_max_tx_num" )

- ( action_validate_exception, chain_exception, 3050000, "Action validate exception" )

- ( account_name_exists_exception, action_validate_exception, 3050001, "Account name already exists" )
- ( invalid_action_args_exception, action_validate_exception, 3050002, "Invalid Action Arguments" )
- ( eosio_assert_message_exception, action_validate_exception, 3050003, "eosio_assert_message assertion failure" )
- ( eosio_assert_code_exception, action_validate_exception, 3050004, "eosio_assert_code assertion failure" )
- ( action_not_found_exception, action_validate_exception, 3050005, "Action can not be found" )
- ( action_data_and_struct_mismatch, action_validate_exception, 3050006, "Mismatch between action data and its struct" )
- ( unaccessible_api, action_validate_exception, 3050007, "Attempt to use unaccessible API" )
- ( abort_called, action_validate_exception, 3050008, "Abort Called" )
- ( inline_action_too_big, action_validate_exception, 3050009, "Inline Action exceeds maximum size limit" )

- ( database_exception, chain_exception, 3060000, "Database exception" )

- ( permission_query_exception,     database_exception, 3060001, "Permission Query Exception" )
- ( account_query_exception,        database_exception, 3060002, "Account Query Exception" )
- ( contract_table_query_exception, database_exception, 3060003, "Contract Table Query Exception" )
- ( contract_query_exception,       database_exception, 3060004, "Contract Query Exception" )

- ( guard_exception, database_exception, 3060100, "Database exception" )

- ( database_guard_exception, guard_exception, 3060101, "Database usage is at unsafe levels" )
- ( reversible_guard_exception, guard_exception, 3060102, "Reversible block log usage is at unsafe levels" )

- ( wasm_exception, chain_exception, 3070000, "WASM Exception" )
- ( page_memory_error,        wasm_exception, 3070001, "Error in WASM page memory" )
- ( wasm_execution_error,     wasm_exception, 3070002, "Runtime Error Processing WASM" )
- ( wasm_serialization_error, wasm_exception, 3070003, "Serialization Error Processing WASM" )
- ( overlapping_memory_error, wasm_exception, 3070004, "memcpy with overlapping memory" )
- ( binaryen_exception, wasm_exception, 3070005, "binaryen exception" )


- ( resource_exhausted_exception, chain_exception, 3080000, "Resource exhausted exception" )

- ( ram_usage_exceeded, resource_exhausted_exception, 3080001, "Account using more than allotted RAM usage" )
- ( tx_net_usage_exceeded, resource_exhausted_exception, 3080002, "Transaction exceeded the current network usage limit imposed on the transaction" )
- ( block_net_usage_exceeded, resource_exhausted_exception, 3080003, "Transaction network usage is too much for the remaining allowable usage of the current block" )
- ( tx_cpu_usage_exceeded, resource_exhausted_exception, 3080004, "Transaction exceeded the current CPU usage limit imposed on the transaction" )
- ( block_cpu_usage_exceeded, resource_exhausted_exception, 3080005, "Transaction CPU usage is too much for the remaining allowable usage of the current block" )
- ( deadline_exception, resource_exhausted_exception, 3080006, "Transaction took too long" )
- ( greylist_net_usage_exceeded, resource_exhausted_exception, 3080007, "Transaction exceeded the current greylisted account network usage limit" )
- ( greylist_cpu_usage_exceeded, resource_exhausted_exception, 3080008, "Transaction exceeded the current greylisted account CPU usage limit" )
- ( leeway_deadline_exception, deadline_exception, 3081001, "Transaction reached the deadline set due to leeway on account CPU limits" )

- ( authorization_exception, chain_exception, 3090000, "Authorization exception" )
- ( tx_duplicate_sig,             authorization_exception, 3090001, "Duplicate signature included" )
- ( tx_irrelevant_sig,            authorization_exception, 3090002, "Irrelevant signature included" )
- ( unsatisfied_authorization,    authorization_exception, 3090003, "Provided keys, permissions, and delays do not satisfy declared authorizations" )
- ( missing_auth_exception,       authorization_exception, 3090004, "Missing required authority" )
- ( irrelevant_auth_exception,    authorization_exception, 3090005, "Irrelevant authority included" )
- ( insufficient_delay_exception, authorization_exception, 3090006, "Insufficient delay" )
- ( invalid_permission,           authorization_exception, 3090007, "Invalid Permission" )
- ( unlinkable_min_permission_action, authorization_exception, 3090008, "The action is not allowed to be linked with minimum permission" )
- ( invalid_parent_permission,           authorization_exception, 3090009, "The parent permission is invalid" )

- ( misc_exception, chain_exception, 3100000, "Miscellaneous exception" )

- ( rate_limiting_state_inconsistent,       misc_exception, 3100001, "Internal state is no longer consistent" )
- ( unknown_block_exception,                misc_exception, 3100002, "Unknown block" )
- ( unknown_transaction_exception,          misc_exception, 3100003, "Unknown transaction" )
- ( fixed_reversible_db_exception,          misc_exception, 3100004, "Corrupted reversible block database was fixed" )
- ( extract_genesis_state_exception,        misc_exception, 3100005, "Extracted genesis state from blocks.log" )
- ( subjective_block_production_exception,  misc_exception, 3100006, "Subjective exception thrown during block production" )
- ( multiple_voter_info,                    misc_exception, 3100007, "Multiple voter info detected" )
- ( unsupported_feature,                    misc_exception, 3100008, "Feature is currently unsupported" )
- ( node_management_success,                misc_exception, 3100009, "Node management operation successfully executed" )



- ( plugin_exception, chain_exception, 3110000, "Plugin exception" )

- ( missing_chain_api_plugin_exception,           plugin_exception, 3110001, "Missing Chain API Plugin" )
- ( missing_wallet_api_plugin_exception,          plugin_exception, 3110002, "Missing Wallet API Plugin" )
- ( missing_history_api_plugin_exception,         plugin_exception, 3110003, "Missing History API Plugin" )
- ( missing_net_api_plugin_exception,             plugin_exception, 3110004, "Missing Net API Plugin" )
- ( missing_chain_plugin_exception,               plugin_exception, 3110005, "Missing Chain Plugin" )
- ( plugin_config_exception,               plugin_exception, 3110006, "Incorrect plugin configuration" )


- ( wallet_exception, chain_exception, 3120000, "Wallet exception" )

- ( wallet_exist_exception,            wallet_exception, 3120001, "Wallet already exists" )
- ( wallet_nonexistent_exception,      wallet_exception, 3120002, "Nonexistent wallet" )
- ( wallet_locked_exception,           wallet_exception, 3120003, "Locked wallet" )
- ( wallet_missing_pub_key_exception,  wallet_exception, 3120004, "Missing public key" )
- ( wallet_invalid_password_exception, wallet_exception, 3120005, "Invalid wallet password" )
- ( wallet_not_available_exception,    wallet_exception, 3120006, "No available wallet" )
- ( wallet_unlocked_exception,         wallet_exception, 3120007, "Already unlocked" )
- ( key_exist_exception,               wallet_exception, 3120008, "Key already exists" )
- ( key_nonexistent_exception,         wallet_exception, 3120009, "Nonexistent key" )
- ( unsupported_key_type_exception,    wallet_exception, 3120010, "Unsupported key type" )
- ( invalid_lock_timeout_exception,    wallet_exception, 3120011, "Wallet lock timeout is invalid" )
- ( secure_enclave_exception,          wallet_exception, 3120012, "Secure Enclave Exception" )


- ( whitelist_blacklist_exception,   chain_exception, 3130000, "Actor or contract whitelist/blacklist exception" )

- ( actor_whitelist_exception,    whitelist_blacklist_exception, 3130001, "Authorizing actor of transaction is not on the whitelist" )
- ( actor_blacklist_exception,    whitelist_blacklist_exception, 3130002, "Authorizing actor of transaction is on the blacklist" )
- ( contract_whitelist_exception, whitelist_blacklist_exception, 3130003, "Contract to execute is not on the whitelist" )
- ( contract_blacklist_exception, whitelist_blacklist_exception, 3130004, "Contract to execute is on the blacklist" )
- ( action_blacklist_exception,   whitelist_blacklist_exception, 3130005, "Action to execute is on the blacklist" )
- ( key_blacklist_exception,      whitelist_blacklist_exception, 3130006, "Public key in authority is on the blacklist" )

- ( controller_emit_signal_exception, chain_exception, 3140000, "Exceptions that are allowed to bubble out of emit calls in controller" )
- ( checkpoint_exception,          controller_emit_signal_exception, 3140001, "Block does not match checkpoint" )


- ( abi_exception, chain_exception, 3015000, "ABI exception" )
- ( abi_not_found_exception,              abi_exception, 3015001, "No ABI found" )
- ( invalid_ricardian_clause_exception,   abi_exception, 3015002, "Invalid Ricardian Clause" )
- ( invalid_ricardian_action_exception,   abi_exception, 3015003, "Invalid Ricardian Action" )
- ( invalid_type_inside_abi,           abi_exception, 3015004, "The type defined in the ABI is invalid" ) // Not to be confused with abi_type_exception
- ( duplicate_abi_type_def_exception,     abi_exception, 3015005, "Duplicate type definition in the ABI" )
- ( duplicate_abi_struct_def_exception,   abi_exception, 3015006, "Duplicate struct definition in the ABI" )
- ( duplicate_abi_action_def_exception,   abi_exception, 3015007, "Duplicate action definition in the ABI" )
- ( duplicate_abi_table_def_exception,    abi_exception, 3015008, "Duplicate table definition in the ABI" )
- ( duplicate_abi_err_msg_def_exception,  abi_exception, 3015009, "Duplicate error message definition in the ABI" )
- ( abi_serialization_deadline_exception, abi_exception, 3015010, "ABI serialization time has exceeded the deadline" )
- ( abi_recursion_depth_exception,        abi_exception, 3015011, "ABI recursive definition has exceeded the max recursion depth" )
- ( abi_circular_def_exception,           abi_exception, 3015012, "Circular definition is detected in the ABI" )
- ( unpack_exception,                     abi_exception, 3015013, "Unpack data exception" )
- ( pack_exception,                     abi_exception, 3015014, "Pack data exception" )

- ( contract_exception,           chain_exception, 3160000, "Contract exception" )
- ( invalid_table_payer,             contract_exception, 3160001, "The payer of the table data is invalid" )
- ( table_access_violation,          contract_exception, 3160002, "Table access violation" )
- ( invalid_table_iterator,          contract_exception, 3160003, "Invalid table iterator" )
- ( table_not_in_cache,          contract_exception, 3160004, "Table can not be found inside the cache" )
- ( table_operation_not_permitted,          contract_exception, 3160005, "The table operation is not allowed" )
- ( invalid_contract_vm_type,          contract_exception, 3160006, "Invalid contract vm type" )
- ( invalid_contract_vm_version,          contract_exception, 3160007, "Invalid contract vm version" )
- ( set_exact_code,          contract_exception, 3160008, "Contract is already running this version of code" )
- ( wast_file_not_found,          contract_exception, 3160009, "No wast file found" )
- ( abi_file_not_found,          contract_exception, 3160010, "No abi file found" )

- ( producer_exception,           chain_exception, 3170000, "Producer exception" )
- ( producer_priv_key_not_found,   producer_exception, 3170001, "Producer private key is not available" )
- ( missing_pending_block_state,   producer_exception, 3170002, "Pending block state is missing" )
- ( producer_double_confirm,       producer_exception, 3170003, "Producer is double confirming known range" )
- ( producer_schedule_exception,   producer_exception, 3170004, "Producer schedule exception" )
- ( producer_not_in_schedule,      producer_exception, 3170006, "The producer is not part of current schedule" )

- ( reversible_blocks_exception,           chain_exception, 3180000, "Reversible Blocks exception" )
- ( invalid_reversible_blocks_dir,             reversible_blocks_exception, 3180001, "Invalid reversible blocks directory" )
- ( reversible_blocks_backup_dir_exist,          reversible_blocks_exception, 3180002, "Backup directory for reversible blocks already existg" )
- ( gap_in_reversible_blocks_db,          reversible_blocks_exception, 3180003, "Gap in the reversible blocks database" )

- ( block_log_exception, chain_exception, 3190000, "Block log exception" )
- ( block_log_unsupported_version, block_log_exception, 3190001, "unsupported version of block log" )
- ( block_log_append_fail, block_log_exception, 3190002, "fail to append block to the block log" )
- ( block_log_not_found, block_log_exception, 3190003, "block log can not be found" )
- ( block_log_backup_dir_exist, block_log_exception, 3190004, "block log backup dir already exists" )

- ( http_exception, chain_exception, 3200000, "http exception" )
- ( invalid_http_client_root_cert,    http_exception, 3200001, "invalid http client root certificate" )
- ( invalid_http_response, http_exception, 3200002, "invalid http response" )
- ( resolved_to_multiple_ports, http_exception, 3200003, "service resolved to multiple ports" )
- ( fail_to_resolve_host, http_exception, 3200004, "fail to resolve host" )
- ( http_request_fail, http_exception, 3200005, "http request fail" )
- ( invalid_http_request, http_exception, 3200006, "invalid http request" )

- ( resource_limit_exception, chain_exception, 3210000, "Resource limit exception" )

- ( mongo_db_exception, chain_exception, 3220000, "Mongo DB exception" )
- ( mongo_db_insert_fail, mongo_db_exception, 3220001, "Fail to insert new data to Mongo DB" )
- ( mongo_db_update_fail, mongo_db_exception, 3220002, "Fail to update existing data in Mongo DB" )

- ( contract_api_exception,    chain_exception, 3230000, "Contract API exception" )
- ( crypto_api_exception,   contract_api_exception, 3230001, "Crypto API Exception" )
- ( db_api_exception,       contract_api_exception, 3230002, "Database API Exception" )
- ( arithmetic_exception,   contract_api_exception, 3230003, "Arithmetic Exception" )
