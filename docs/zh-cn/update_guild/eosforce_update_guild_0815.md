# 2018年8月10日BP节点预演说明

因EOSFORCE要上线新版本，提供更加可靠健壮，功能丰富的eosforce主网，本次预演由当前竞选BP节点社区共同完成一次同一个主网的升级预演，需要各BP节点社区配合原力这边的技术支持，更新自己BP节点服务，即停服务，替换成由原力提供最新代码编译出来的可执行程序来启动主网

## 事项和规则
1. 旧版本eosforce主网代码和当前线上eosforce主网代码一致，作为测试环境， 新版本eosforce主网代码是原力官方即将升级的的eosforce主网代码

2. 本次主网升级预演完全在测试环境进行，不需要动已运行在线上的eosforce主网上BP节点

3. 升级由BP节点一个一个按固定顺序（23个BP节点按节点投票排名由高到低的顺序来完成升级），前一BP升级成功，下一个BP节点开始完成升级，确保主网最后预演成功

4.在预演升级之前，只需要把旧版本eosforce主网部署好，到时直接演练升级为新版本eosforce主网

5.本次预演环境的原力官方钱包下载地址为 https://github.com/eosforce/wallet-desktop/releases/tag/v1.0.57

## 预演流程

本次预演升级是基于测试环境进行的，为了后面更顺利更安全的完成线上eosforce主网系统的升级，本次预演需要每个BP节点完成以下7个步骤：

1. 准备工作： 这次节点升级预演基于在原力eos主网测试环境下进行，各bp社区的需要准备一台测试机器（配置为ubuntu 16.04 64位linux 操作系统，配置2核4G内存以上）作为这次预演的测试机器
 
2.  旧版本eosforce主网部署： 旧版本bp节点和当前线上eosforce代码一样， 需要各社区在搭建一套测试环境模拟真实的线上，具体是现役23个超级节点在已准备好的测试机器上基于旧版本的eosforce的代码来重新部署一次bp节点，这样旧版本bp节点完成部署（需完成节点部署，bp节点注册）

3. 旧版本eosforce主网测试： 基于部署好的后各bp节点开始进行出块，投票等其他相关测试

4. 停止旧版本eosforce主网服务： 测试如果通过，停止旧版本的nodeos进程服务并进行下一步，否则退回上一步

5. 新版本eosforce主网的升级：基于上一步已停掉的旧版本的eosforce主网来完成升级，具体是由我们提供的最新eosforce的代码来编译生成的可执行文件替换旧版本的可执行文件，启动服务完成升级（只需替换编译出来的可执行文件）

6. 新版本eosforce主网测试： 各bp节点开始进行 出块，投票等其他相关测试

7. 预演结束： 测试如果通过这次预演成功结束，否则退回上一步


## 旧版本eosforce主网部署说明

请参考这个文档来部署节点  http://47.92.77.94:4000/eosforce-old.html

注意这个部署文档和之前部署正式节点比较类似，有2处不一样，genesis.json文件修改了以及config.ini配置p2p-peer-address选项不一样，在文档中p2p-peer-address只配置了一个（47.98.249.86:8002）， 请认真阅读文档

其中docker部署也一样p2p-peer-address也要配置成 47.98.249.86:8002 这个地址


## 使用源码编译升级部署说明

因新版本中增加了eosio.msig， eosio.bios相关合约，因此在部署时也有差别，注意下面的文档

下面的升级操作都在一台测试机上进行，即升级前的主网部署在这台机器上，需要升级的新版本源码编译也在这台机器上

这里以编译部署为例子来说明，这里假定升级前的源码目录是/root/eosforce，需要升级的源码目录为/root/eosforce-new


### 1. 下载源码
新版本代码在release分支中，注意切换分支
```shell
apt-get update && apt-get install -y git wget
git clone https://github.com/eosforce/eosforce.git eosforce-new
cd eosforce-new
git checkout -b release
git pull origin release
```

### 2. 执行如下命令进行编译

```shell
git submodule update --init --recursive && ./eosio_build.sh
```

### 3. 编译出来后相对于之前版本会多出来 eosio.bios.abi，eosio.bios.wasm，eosio.msig.abi，eosio.msig.wasm 4个智能合约文件，需要将新增的智能文件拷贝进之前存放合约的目录

要确保旧版本的 ~/.local/share/eosio/nodeos/config目录已有System.abi,System.wasm,config.ini	,eosio.token.abi,eosio.token.wasm,genesis.json这些文件

```shell
cp build/contracts/eosio.bios/eosio.bios.abi build/contracts/eosio.bios/eosio.bios.wasm ~/.local/share/eosio/nodeos/config
cp build/contracts/eosio.msig/eosio.msig.abi build/contracts/eosio.msig/eosio.msig.wasm ~/.local/share/eosio/nodeos/config
cd build && make install
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

来部署升级前的主网，注意conifg.ini配置，其p2p-peer-address要配置成 47.98.249.86:8002,目前只提供一个地址

部署完成后需要确保/data/eosforce下有System.abi，System.wasm, eosio.bios.abi, eosio.bios.wasm,eosio.msig.abi,eosio.msig.wasm,eosio.token.abi,eosio.token.wasm,genesis.json,config.ini文件即可

这里升级前版本是eosforce/eos:v1.0.0，需要升级的版本为eosforce/eos:v1.1

我们提供了eosforce/eos:v1.0.0和eosforce/eos:v1.1镜像

 ```shell
 docker pull eosforce/eos:v1.0.0
 docker pull eosforce/eos:v1.1
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

