# Theme

This document describes how to use docker to deploy eosforce-v1.8.3 node.

## What to watch out for when upgrading BP

DO NOT upgrade the BP node directly. Upgrade the synchronization node first. After the synchronization node has been upgraded to v-1.8.3, migrate the BP node.


In v-1.8.3, heartbeat_plugin is no longer integrated. You need to remove heartbeat_plugin from config.ini.

**Node synchronization can be done by quickly starting the snapshot. If other nodes(including BP nodes) are started with the data of the synchronization node, use the normal method of starting the eosforce node.**

## Deployment steps

### Pull Docker image of v1.8.3

```bash
docker pull eosforce/node:v1.8.3
```

### Prepare external files for docker startup

It is recommended that when using Docker to start, the configuration file and data file are placed on the host machine to facilitate the host machine to modify and read. The files that need to be placed on the host machine are

| Folder Name | Description                                                                                                      |
|:-----------|:----------------------------------------------------------------------------------------------------------|
| config     | The config folder stores the configuration files for node startup, including config.ini genesis.json and related abi and wasm files that need to be loaded when the chain starts |
| data       | The data folder stores blockchain data and is placed outside the host machine for easy backup and deletion.                                             |
| snapshots  | V1.8.3 supports starting from snapshots, and the snapshots folder stores related snapshot files                                          |

Directory Format

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

[download eosforce snapshot](https://github.com/awaketeam/eosc_snapshots)


### Start eosforce node via docker

#### Start eosforce node via snapshot

```bash
docker run -d --name eosforce-snapshot -v /docker_data:/eosforce -p 39876:39876 -p 38889:38889 eosforce/node:v1.8.3 /opt/eosio/bin/nodeos --config-dir=/eosforce/config --data-dir=/eosforce/data --snapshot=/eosforce/snapshots/snapshot-*.bin
```

Related parameter description
| Name   | Description                                    | Supplementary note                                                                                                            |
|:-------|:----------------------------------------|:--------------------------------------------------------------------------------------------------------------------|
| -d     | Specifies whether the container runs in the foreground or background. The default is false. | Run node in the background                                                                                                |
| --name | Name the container                            | --name eosforce-snapshot is to name the container as eosforce-snapshot                                                            |
| -v     | Mount the storage volume to the container and mount it to a directory in the container  | -v / docker_data: / eosforce is to mount the host's / docker_data directory to / eosforce inside docker, this directory is shared by the host and docker |
| -p     | Specify the port exposed by the container                      | -p 39876: 39876 is to mount docker's 39876 port to the host's 39876 port, so that you can access docker's 39876 port through the host's 39876 port     |
|/opt/eosio/bin/nodeos --config-dir=/eosforce/config --data-dir=/eosforce/data --snapshot=/eosforce/snapshots/snapshot-*.bin | The command to start the node node specifies three parameters --config-dir, --data-dir, and --snapshot. The address requires the directory address in docker.||

**When startup from a snapshot, you need to configure produce_plugin and produce_api_plugin on config.ini. The data folder specifies an empty folder.**

```config.ini
plugin = eosio::producer_plugin
plugin = eosio::producer_api_plugin
```

#### Shut down the eosforce node normally

```bash
docker stop eosforce-snapshot
```

After a normal shutdown, the data file can be used by other 1.8.x versions of eosforce nodes. When using it, you need to use the normal method of starting eosforce nodes.

#### Start eosforce node normally

```bash
docker run -d --name eosforce-v1.8.3 -v /docker_data:/eosforce -p 39876:39876 -p 38889:38889 eosforce/node:v1.8.3 /opt/eosio/bin/nodeos --config-dir=/eosforce/config --data-dir=/eosforce/data
```

It can be seen from the command that the normal startup has one less --snapshot parameter than the snapshot startup, and the relevant parameter description is not repeated here.



## Ending

It is very convenient to upgrade nodes through docker. Because eosforce v1.8.3 supports snapshot function, you can quickly restore a node through snapshot function. BP nodes are recommended to be upgraded to v1.8.3 and above.