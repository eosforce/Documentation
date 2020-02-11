# 主题

此文档介绍如何使用docker部署eosforce-v1.8.3节点

## BP节点升级需要注意

BP节点的升级不要直接升级，先升级同步节点，同步节点升级1.8.3同步区块完毕以后再把出块节点迁移过去。

1.8.3版本不再集成heatbeat_plugin插件，需要再config.ini上面把heatbneat_plugin插件去掉。

**同步节点可以通过快照快速启动完成，如果其他节点（包括出块节点）使用同步节点的data数据启动的话使用正常启动eosforce节点的方法**

## 部署步骤

### 拉取1.8.3版本Docker镜像

```bash
docker pull eosforce/node:v1.8.3
```

### 准备docker启动时使用的外部文件

建议使用Docker启动的时候，配置文件和数据文件都放到宿主机上面，方便宿主机进行修改和读取。需要放置在宿主机上面的文件有

| 文件夹名称 | 描述                                                                                                      |
|:-----------|:----------------------------------------------------------------------------------------------------------|
| config     | config文件夹存放节点启动的配置文件，包括config.ini  genesis.json  以及链启动时需要加载的相关abi和wasm文件 |
| data       | data文件夹存放区块链数据，放到宿主机外部便于及时备份和删除。                                              |
| snapshots  | 1.8.3版本支持通过快照启动，snapshots文件夹存放相关快照文件                                                |

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

#### 通过快照启动eosforce节点

```bash
docker run -d --name eosforce-snapshot -v /docker_data:/eosforce -p 39876:39876 -p 38889:38889 eosforce/node:v1.8.3 /opt/eosio/bin/nodeos --config-dir=/eosforce/config --data-dir=/eosforce/data --snapshot=/eosforce/snapshots/snapshot-*.bin
```

相关参数说明
| 名称   | 描述                                    | 补充说明                                                                                                            |
|:-------|:----------------------------------------|:--------------------------------------------------------------------------------------------------------------------|
| -d     | 指定容器运行于前台还是后台，默认为false | 将node节点在后台运行                                                                                                |
| --name | 给容器命名                              | --name eosforce-snapshot 是给容器起名为eosforce-snapshot                                                            |
| -v     | 给容器挂载存储卷，挂载到容器的某个目录  | -v /docker_data:/eosforce 是将宿主机的/docker_data目录挂载到docker里面的/eosforce，这个目录就是宿主机和docker共享的 |
| -p     | 指定容器暴露的端口                      | -p 39876:39876 将docker的39876端口挂载到宿主机的39876端口，这样可以通过宿主机的39876端口访问到docker的39876端口     |
|/opt/eosio/bin/nodeos --config-dir=/eosforce/config --data-dir=/eosforce/data --snapshot=/eosforce/snapshots/snapshot-*.bin | 启动node节点的命令，这里面指定了--config-dir，--data-dir，--snapshot 三个参数，地址需要时docker里面目录地址。||

**通过快照启动的时候需要config.ini上面配置produce_plugin和produce_api_plugin,data文件夹指定一个空的文件夹**

```config.ini
plugin = eosio::producer_plugin
plugin = eosio::producer_api_plugin
```

#### 正常关闭eosforce节点

```bash
docker stop eosforce-snapshot
```

正常关闭以后data文件可以给其他1.8.x版本的eosforce节点使用，使用时需要用正常启动eosforce节点的方法。

#### 正常启动eosforce节点

```bash
docker run -d --name eosforce-v1.8.3 -v /docker_data:/eosforce -p 39876:39876 -p 38889:38889 eosforce/node:v1.8.3 /opt/eosio/bin/nodeos --config-dir=/eosforce/config --data-dir=/eosforce/data
```

由命令可以看出正常启动比快照启动少了--snapshot  参数，相关参数说明不再赘述。



## 结束语

通过docker升级节点还是很方便的，由于eosforce1.8.3版本支持快照功能，通过快照功能可以快速恢复一个节点，建议出块节点都能升级到1.8.3及以上的版本。