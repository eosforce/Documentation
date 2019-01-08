# Cleos system说明

system是cleos的一个基本命令，有更新bp，设置/取消链紧急状态等功能


## 测试链

- IP: 127.0.0.1

- Port：8888

使用示例: 

```bash
cleos -u http://127.0.0.1:8888 get info
```

## cleos system 命令参数

cleos system [选项] 子命令
```
选项:
  -h,--help                   打印帮助信息

Subcommands:
  updatebp                        更新bp信息
  setemergency                    设置链紧急状态
  cancleemergency                 取消链紧急状态
  listbps                    	  列出所有的bp节点
  canceldelay			  取消延迟的交易
```
## cleos voteproducer 子命令详解
### updatebp
更新bp的信息
cleos system updatebp [选项] account producer_key commission_rate url
  account			bp节点的账户
  producer_key			bp节点的公钥
  commission_rate		bp节点用于从投票收益上获取收益的比例（0-10000）
  url				bp节点的连接地址
使用示例: 
更新bp节点biosbpa的信息
```bash
cleos system updatebp biosbpa EOS6gBsQtMe5Ce5nkoE2Dx4dSkLu2EbodUeL8AWcqJ4mR8SX1XtSD 100 www.biosbpa.com
```
### setemergency
设置链紧急状态，用于在特殊情况下使用，一旦2/3的出块节点都设置的紧急状态，该链上的交易、投票等操作将无法执行。一般用于大的升级前使用
cleos system setemergency [选项] bp_name
  bp_name 			bp的名称
使用示例: 
biosbpa将节点设置为链紧急状态
```bash
cleos system setemergency biosbpa
```
注意：
>该状态一般在可能造成分叉的升级前设置，只有出块节点设置才有效，升级之后需要设置回来
### cancleemergency 
取消链上紧急状态
cleos system cancleemergency [选项] bp_name
  bp_name 			bp的名称
使用示例: 
biosbpa取消链紧急状态
```bash
cleos system cancleemergency biosbpa
```
### listbps
列出当前所有的bp信息
cleos system listbps [选项]
  
使用示例: 
```bash
cleos system listbps
```
### canceldelay
取消一个延迟的事务
cleos system canceldelay [选项] canceling_account canceling_permission trx_id
canceling_account	取消的延迟事务的账户
canceling_permission	取消的延迟事务的权限
trx_id			取消延迟事务的transaction_id
使用示例: 42fe264a09f280002746939c3083e98d6362eac0b5339c7b779bdbadc907c286
```bash
cleos system canceldelay eosforce active 42fe264a09f280002746939c3083e98d6362eac0b5339c7b779bdbadc907c286
```


