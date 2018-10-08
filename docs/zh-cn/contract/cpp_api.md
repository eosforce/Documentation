# C 编程接口

Action

定义查询和发送 action 编程接口。

EOSFORCE.IO action 拥有下面的抽象结构：

	struct action {
  		scope_name scope; // the contract defining the primary code to execute for code/type
  		action_name name; // sction 名称
  		permission_level[] authorization; // 账户和权限级别
  		bytes data; // 代码处理的不透明的数据
	};

这个接口时合约能检视当前 action 及 act 中字段。

例如：

	// Assume this action is used for the following examples:
	// {
	//  "code": "eos",
	//  "type": "transfer",
	//  "authorization": [{ "account": "inita", "permission": "active" }],
	//  "data": {
	//    "from": "inita",
	//    "to": "initb",
	//    "amount": 1000
	//  }
	// }
	
	char buffer[128];
	uint32_t total = read_action_data(buffer, 5); // buffer 包含最多5字节 action 内容
	print(total); // 输出: 5
	
	uint32_t msgsize = action_data_size();
	print(msgsize); // 输出: 上述 action data 字段的大小
	
	require_recipient(N(initc)); // initc 账户对这个 action 将被通知
	
	require_auth(N(inita)); // 不做任何事因为 inita 存在于认证列表中
	require_auth(N(initb)); // 抛异常
	
	print(current_time()); // 输出: 当前块的时间戳（自从1970年以来的微妙数）
                   
 总结

	成员 																	描述
	public uint32_t read_action_data (void * msg,uint32_t len) 				拷贝当前 action 数据到指定位置。
	public uint32_t action_data_size() 										获取当前 action 数据长度。
	public void require_recipient(account_name name) 						增加指定账户到被通知账户集合。
	public void require_auth(account_name name) 							验证指定账户是否存在于指定的授权集合中。
	public bool has_auth(account_name name) 								验证action 账户集合是否包含指定账户
	public void require_auth2(account_name name,permission_name permission) 验证指定账户是否存在于指定的授权集合中。
	public bool is_account(account_name name) 	
	public void send_inline(char * serialized_action,size_t size) 			在当前 action 的父transaction 上下文中发送在线 action 。
	public void send_context_free_inline(char * serialized_action,size_t size) 	在当前 action 的父transaction 上下文中发送上下文无关在线 action 。
	public void require_write_lock(account_name name) 	Verifies that name exists in the set of write locks held.
	public void require_read_lock(account_name name) 						验证 name 是否存在于持有的读锁集合中。
	public uint64_t publication_time() 										获取出版时间。
	public account_name`[current_receiver](#current_receiver)()` 			获取 action 当前接收者。      
                   
                   
read_action_data                  
拷贝当前 action 数据到指定位置。              

	uint32_t read_action_data( void* msg, uint32_t len );

拷贝最多 len 字节当前 action 数据到指定位置。    
 
参数
msg - 指向保存 action 数据的位置
len - 要拷贝的数据的长度，0 报告需要的长度

返回值
拷贝到指定位置的字节数，或者能拷贝的字节数（如果len传0） 

前置条件
msg 是有效指针指向至少 len 字节长度的内存区域

后置条件
msg 被填充打包的 action 数据

                   
action_data_size
获取当前 action 数据长度。

	uint32_t action_data_size();

获取当前 action 数据长度。这个方法对动态大小的 actions 有用。

返回值
当前 action 数据字段的长度
 
 
require_recipient
 
 
 
 
 
 
 
 
 
 
 
 
 
 
                   
                   
# C++ 编程接口















# 类型





