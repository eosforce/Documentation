# Cleos wallet说明

wallet是cleos操作钱包的一个命令


## 测试链

- IP: 127.0.0.1

- Port：8888

使用示例: 

```bash
cleos -u http://127.0.0.1:8888 get info
```

## cleos wallet 命令参数

cleos wallet [选项] 子命令
```
选项:
  -h,--help                   打印帮助信息

Subcommands:
  create                        创建一个新的钱包
  open                    	打开一个钱包
  lock                 		锁定一个钱包
  lock_all                    	锁定所有钱包
  unlock			解锁一个钱包
  import			在钱包里导入一个私钥
  remove_key			在钱包里移除一个公钥
  create_key			创建一个私钥并导入到钱包中
  list				列出所有的钱包
  keys				列出钱包的所有的公钥
  private_keys			列出钱包的所有公钥私钥对
  stop				停止keosd
```
## cleos wallet 子命令详解
### create
创建一个新的钱包
cleos wallet create [选项]
选项:
  -n,--name			新钱包的名字
  -f,--file			钱包的密码输出的文件名
  --to-console			打印钱包密码到shell上面
使用示例: 
创建一个叫test1的钱包
```bash
cleos wallet create -n test1 --to-console
```
### open
打开一个钱包
cleos wallet open [选项]
选项:
  -n,--name 			打开的钱包的名字
使用示例: 
打开钱包test1
```bash
cleos wallet open -n test1
```
### lock 
锁定一个钱包
cleos wallet lock [选项]
选项:
  -n,--name 			打开的钱包的名字
使用示例: 
锁定钱包test1
```bash
cleos wallet lock -n test1
```
### lock_all
列出当前所有的bp信息
cleos system lock_all
  
使用示例: 
```bash
cleos system lock_all
```
### unlock
解锁一个钱包
cleos wallet unlock [选项]
选项：
-n,--name 			解锁的钱包的名字
--password			解锁钱包的密码
使用示例:
```bash
cleos wallet unlock -n test1 --password  PW5K8YT1TTGZTAjJYVoQ8DstrrEuDXamx6wGaAYjK5poT2CkqyFd1
```
### import
导入私钥到钱包里
cleos wallet import [选项]
选项：
-n,--name 			需要导入私钥的钱包的名字
--private-key			导入的私钥
使用示例:
```bash
cleos wallet import -n test1 --private-key 5KhfDXX2iXR54ktXVuuCC7a4UJxAZaSxnH4BnEfuMRbBMH576TW
```
### remove_key
将一个公钥从钱包里移除
cleos wallet remove_key [选项] key
key				移除的公钥
选项：
-n,--name 			钱包的名字
--password			钱包的密码
使用示例:
```bash
cleos wallet  remove_key EOS5hJuWr6DdjX2xgJbMZxH499ZkgenEgB3wbhsXKZgbGzEFfvYgz -n test1 --password PW5K8YT1TTGZTAjJYVoQ8DstrrEuDXamx6wGaAYjK5poT2CkqyFd1
```
### create_key
创建一个私钥公钥对并导入到钱包中
cleos wallet create_key [OPTIONS] [key_type]
选项：
-n,--name 			钱包的名字
使用示例:
```bash
cleos wallet create_key -n test1
```
### list
列出所有的钱包
cleos wallet list
使用示例:
```bash
cleos wallet list
```
### keys
列出所有的公钥
cleos wallet keys
```bash
cleos wallet keys
```
### private_keys
列出所有的公钥私钥对
cleos wallet private_keys [选项]
-n,--name 			钱包的名字
--password			钱包的密码
使用示例:
```bash
cleos wallet private_keys -n test1 --password PW5K8YT1TTGZTAjJYVoQ8DstrrEuDXamx6wGaAYjK5poT2CkqyFd1
```
### stop
停止keosd
cleos wallet stop
使用示例:
```bash
cleos wallet stop
```
