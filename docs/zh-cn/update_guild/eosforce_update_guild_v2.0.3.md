# v2.0.3节点更新操作步骤

## 升级内容

合并EOSIO 2.0.3版本

**注意:v2.0.3版本的节点不能使用v1.x.x版本的Data数据,建议先使用快照快速启动,然后用v2.0.3版本生成的Data重新启动节点**

## docker部署

```bash
docker pull eosforce/node:v2.0.3
```

### 准备docker启动外部文件

目录格式

```
├── docker_data
│ ├── config
│ │ ├── activeacc.json
│ │ ├── config.ini
│ │ ├── eosio.lock.abi
│ │ ├── ...
│ │ └── genesis.json
│ ├── data
│ ├── snapshots
│ │ ├──snapshot-*.bin
```

[eosforce快照下载地址](https://github.com/awaketeam/eosc_snapshots)

### 通过docker启动eosforce节点

```bash
docker rm eosforce-snapshot

docker run -d --name eosforce-snapshot -v /docker_data:/eosforce -p 9876:9876 -p 8889:8889 eosforce/node:v2.0.3 /opt/eosio/bin/nodeos --config-dir=/eosforce/config --data-dir=/eosforce/data --snapshot=/eosforce/snapshots/snapshot-*.bin
```

### 关闭根据快照启动的节点

```bash
docker stop eosforce-snapshot
```

### 正常启动eosforce节点

```bash

docker run -d --name eosforce-v2.0.3  -v /docker_data:/eosforce -p 9876:9876 -p 8889:8889 eosforce/node:v2.0.3 /opt/eosio/bin/nodeos --config-dir=/eosforce/config --data-dir=/eosforce/data

```

这样就使用eosforce-snapshot生成的Data文件正常启动eosforce节点了.

## 编译源码部署

使用tag:eosc-v2.0.3

### 编译源码

```shell
# 进入eosforce工程目录

git fetch
git checkout eosc-v2.0.3
git submodule update --init --recursive
./scripts/eosio_build.sh

```

注意事项（如果编译出错，需要删除HOME目录下的opt目录，重新编译）

编译后使用 build/bin/下生产的可执行文件：cleos  keosd  nodeos

(启动服务仅使用 nodeos)

###  准备启动节点需要的文件

和准备docker启动外部文件 一样,需要准备config目录及其文件,如果需要快照则需要下载最新快照.此处默认使用相同文件

目录格式

```
├── docker_data
│ ├── config
│ │ ├── activeacc.json
│ │ ├── config.ini
│ │ ├── eosio.lock.abi
│ │ ├── ...
│ │ └── genesis.json
│ ├── data
│ ├── snapshots
│ │ ├──snapshot-*.bin
```

[eosforce快照下载地址](https://github.com/awaketeam/eosc_snapshots)

### 快照方式启动

```bash
# 进入eosforce工程目录
./build/bin/nodeos --config-dir  /docker_data/config  --data-dir=/docker_data/data --snapshot=/docker_data/snapshot-*.bin

```

可以使用Ctrl+C来结束node

### 正常启动

```shell
# 进入eosforce工程目录
# 启动
nohup ./build/bin/nodeos --config-dir=/docker_data/config --data-dir =/docker_data/data  > eos.log 2>&1 &

# 查看日志，观察同步或出块是否正常
tail -100f eos.log
```
