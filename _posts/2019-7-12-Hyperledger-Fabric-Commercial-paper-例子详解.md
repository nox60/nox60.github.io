# 本文目的

# 环境设置

要完成本文的所有人物，需要准备以下环境：

## Node

Node版本8.9.0以上

到：https://nodejs.org/download/release/ 下载一个稳定版本的nodejs

```downloadnodejs
wget https://nodejs.org/download/release/latest-v10.x/node-v10.16.0-linux-x64.tar.gz
```

解压

```unzip
tar -xzvf node-v10.16.0-linux-x64.tar.gz
```
一般来说你会把解压后的目录做一些自己的安排，比如改名，移动到你指定的某个目录下，此处是放在了 /root/bin/nodejs1016 这个目录下面。

```setpath
# ========== Nodejs settings =====================
export PATH=/root/bin/node1016/bin:$PATH
```

加入环境变量之后，通过下面的命令验证node的安装是否成功
```nodejs
[root@192-168-106-1 node1016]# node -v
v10.16.0
```
能正确显示版本号，说明node安装成功


## Docker

本Demo对Docker版本的要求是高于18.06

TD: Docker安装


## Golang

TD：Golang安装

$GOPATH is an important environment variable in Hyperledger Fabric; it identifies the root directory for installation. It is important to get right no matter which programming language you’re using! Open a new terminal window and check your $GOPATH is set using the env command:

# 安装程序

## 拉取最新代码

```clonecode
git clone https://github.com/hyperledger/fabric-samples.git
```

进入到 fabric-samples/commercial-paper 目录，这个目录是commercial-paper这个demo项目的目录所在

接下来将会做以下任务来演示该程序：

- to run applications on behalf of Isabella and Balaji who will trade commercial paper with each other

- to issue commands to on behalf of administrators from MagnetoCorp and DigiBank, including installing and instantiating smart contracts

- to show peer, orderer and CA log output

## 创建网络

The tutorial currently uses the basic network; it will be updated soon to a configuration which better reflects the multi-organization structure of PaperNet. For now, this network is sufficient to show you how to develop an application and smart contract.

目前看到了这一页

基本的网络构成：

1. 一个节点peer。
2. 一个账本数据库ledger。
3. 一个排序节点orderer。
4. 一个证书签发机构CA。

在本例中，这些节点都是以容器的方式运行。在生产环境中，一般使用共享的CA。 they’re not dedicated to the Fabric network.（TD）

网络相关的配置代码在 fabric-samples/basic-network 目录下，如果不做任何配置，首先通过 start.sh 脚本启动网络，看下面的示例：

```exampleofstartnetwork

[root@192-168-166-1 basic-network]# ./start.sh

# don't rewrite paths for Windows Git Bash users
export MSYS_NO_PATHCONV=1

docker-compose -f docker-compose.yml down
Removing network net_basic
WARNING: Network net_basic not found.

docker-compose -f docker-compose.yml up -d ca.example.com orderer.example.com peer0.org1.example.com couchdb
Creating network "net_basic" with the default driver
Creating couchdb             ... done
Creating ca.example.com      ... done
Creating orderer.example.com ... done
Creating peer0.org1.example.com ... done
docker ps -a
CONTAINER ID        IMAGE                        COMMAND                  CREATED                  STATUS                  PORTS                                            NAMES
e015028ac314        hyperledger/fabric-peer      "peer node start"        Less than a second ago   Up Less than a second   0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
bdcb9cb18086        hyperledger/fabric-orderer   "orderer"                1 second ago             Up Less than a second   0.0.0.0:7050->7050/tcp                           orderer.example.com
48f5341cc65a        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   1 second ago             Up Less than a second   4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp       couchdb
922314e386e4        hyperledger/fabric-ca        "sh -c 'fabric-ca-se…"   1 second ago             Up Less than a second   0.0.0.0:7054->7054/tcp                           ca.example.com

# wait for Hyperledger Fabric to start
# incase of errors when running later commands, issue export FABRIC_START_TIMEOUT=<larger number>
export FABRIC_START_TIMEOUT=10
#echo ${FABRIC_START_TIMEOUT}
sleep ${FABRIC_START_TIMEOUT}

# Create the channel
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel create -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx
2019-07-20 15:17:41.540 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2019-07-20 15:17:41.553 UTC [cli.common] readBlock -> INFO 002 Received block: 0
# Join peer0.org1.example.com to the channel.
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel join -b mychannel.block
2019-07-20 15:17:41.665 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2019-07-20 15:17:41.717 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel


```