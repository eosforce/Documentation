# 主网升级预演操作文档 18-10-17

预演包含以下几个步骤：

- 1. 准备工作： 这次节点升级预演基于在原力eos主网测试环境下进行，各bp社区的需要准备一台测试机器（配置为ubuntu 16.04 64位linux 操作系统，配置2核4G内存以上）作为这次预演的测试机器

- 2. 旧版本eosforce主网部署： 旧版本bp节点和当前线上eosforce代码一样， 需要各社区在搭建一套测试环境模拟真实的线上，具体是现役23个超级节点在已准备好的测试机器上基于旧版本的eosforce的代码来重新部署一次bp节点，这样旧版本bp节点完成部署（需完成节点部署，bp节点注册）

- 3. 旧版本eosforce主网测试： 基于部署好的后各bp节点开始进行出块，投票等其他相关测试

- 4. 停止旧版本eosforce主网服务： 测试如果通过，停止旧版本的nodeos进程服务并进行下一步，否则退回上一步

- 5. 新版本eosforce主网的升级：基于上一步已停掉的旧版本的eosforce主网来完成升级，具体是由我们提供的最新eosforce的代码来编译生成的可执行文件替换旧版本的可执行文件，启动服务完成升级（只需替换编译出来的可执行文件）

- 6. 新版本eosforce主网测试： 各bp节点开始进行 出块，投票等其他相关测试

- 7. 预演结束： 测试如果通过这次预演成功结束，否则退回上一步


## 旧版本eosforce主网部署说明

请参考这个文档来部署节点  http://47.99.138.131:4000/eosforce.html

注意这个部署文档和之前部署正式节点比较类似，有2处不一样，genesis.json文件修改了以及config.ini配置p2p-peer-address选项不一样：

p2p节点如下：

```
p2p-peer-address = 47.99.151.178:9076
p2p-peer-address = 47.99.138.131:9076
p2p-peer-address = 47.99.165.99:9076
p2p-peer-address = 47.99.167.137:9076
```

genesis.json 可以从http://download.aitimeout.site/eosforce.tar.gz 中获取

## 使用源码编译升级部署说明

因新版本中增加了System01合约，因此在部署时也有差别，注意下面的文档

下面的升级操作都在一台测试机上进行，即升级前的主网部署在这台机器上，需要升级的新版本源码编译也在这台机器上

这里以编译部署为例子来说明，这里假定升级前的源码目录是/root/eosforce，需要升级的源码目录为/root/eosforce-new


### 1. 下载源码
新版本代码在release分支中，注意切换分支

```shell
apt-get update && apt-get install -y git wget
git clone https://github.com/eosforce/eosforce.git eosforce-new
cd eosforce-new
```

### 2. 执行如下命令进行编译

```shell
git submodule update --init --recursive && ./eosio_build.sh
```

### 3. 复制系统合约

编译出来后相对于之前版本会多出来 System01.abi，System01.wasm  2个智能合约文件，需要将新增的智能文件拷贝进之前存放合约的目录
要确保旧版本的 ~/.local/share/eosio/nodeos/config目录已有System.abi,System.wasm,config.ini	,eosio.token.abi,eosio.token.wasm,genesis.json这些文件

```shell
cp build/contracts/System01/System01.abi build/contracts/System01/System01.wasm ~/.local/share/eosio/nodeos/config
cd build && make install
```

另外config.ini中需要添加以下配置：

```
http-validate-host=false
System01-contract-block-num=3500
```

### 4. 先将升级前的build目录备份，再用编译出来的build目录拷贝进升级前的源码目录

```shell
mv /root/eosforce/build /root/eosforce/build.bak
cp -r /root/eosforce-new/build /root/eosforce
```

### 5. 杀掉升级前的nodeos进程，重启节点服务

注意这里需用kill -2 来杀掉进程，不要直接用kill -9来杀进程，否则会启动会报错

```shell
kill -2 pid
```

### 6.最后启动升级后的服务

```shell
cd build/programs/nodeos &&  nohup ./nodeos >> nodeos.log 2>&1 &
```


## 使用docker升级部署说明

参考来部署 https://github.com/eosforce/genesis/tree/release
（其中/data/eosforce/里面的合约文件和genesis.json文件即兼容升级前版本的，又兼容升级后的版本）

来部署升级前的主网，注意conifg.ini配置，其p2p-peer-address要配置成地址

p2p节点如下：

```
p2p-peer-address = 47.99.151.178:9076
p2p-peer-address = 47.99.138.131:9076
p2p-peer-address = 47.99.165.99:9076
p2p-peer-address = 47.99.167.137:9076
```

genesis.json System01.abi，System01.wasm 可以从 http://download.aitimeout.site/eosforce.tar.gz

部署完成后需要确保/data/eosforce下有System01.abi，System01.wasm, System.abi，System.wasm, eosio.bios.abi, eosio.bios.wasm,eosio.msig.abi,eosio.msig.wasm,eosio.token.abi,eosio.token.wasm,genesis.json,config.ini文件即可

这里升级前版本是eosforce/eos:v1.0.1，需要升级的版本为eosforce/eos:v1.1.0.test2

我们提供了eosforce/eos:v1.0.1和eosforce/eos:v1.1.0.test2镜像

 ```shell
 docker pull eosforce/eos:v1.0.1
 docker pull eosforce/eos:v1.1.0.test2
 ```

 另外config.ini中需要添加以下配置：

```
http-validate-host=false
System01-contract-block-num=3500
```

 ### 使用如下命令启动的升级前的主网
 
 ```shell
 docker run -d --restart=always --name eosforce -v /data/eosforce:/opt/eosio/bin/data-dir -v /data/nodeos/eosforce:/root/.local/share/eosio/nodeos -p 8888:8888 -p 9876:9876 eosforce/eos:v1.0.0 nodeosd.sh
```

 ### 升级只需要用新镜像启动,数据目录不变，就可升级成功

  ```shell
  docker stop eosforce
  docker run -d --restart=always --name eosforce-new -v /data/eosforce:/opt/eosio/bin/data-dir -v /data/nodeos/eosforce:/root/.local/share/eosio/nodeos -p 8888:8888 -p 9876:9876 eosforce/eos:v1.1 nodeosd.sh
  ```

