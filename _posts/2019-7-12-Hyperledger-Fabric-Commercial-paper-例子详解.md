# 本文目的

# 环境设置

要完成本文的所有任务，需要准备以下环境：

## Node

Node版本8.9.0以上

到：https://nodejs.org/download/release/ 下载一个稳定版本的nodejs，下面选择的是10.16.0版本

```downloadnodejs
wget https://nodejs.org/download/release/latest-v10.x/node-v10.16.0-linux-x64.tar.gz
```

解压

```unzip
tar -xzvf node-v10.16.0-linux-x64.tar.gz
```
一般来说你会把解压后的目录做一些自己的安排，比如改名，移动到你指定的某个目录下，此处是放在了 /root/bin/nodejs1016 这个目录下面。然后设置环境变量，以保证能够运行node程序。

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

另外需要参考下面的帖子对 npm 的全局安装目录进行改动：

https://stackoverflow.com/questions/29468404/gyp-warn-eacces-user-root-does-not-have-permission-to-access-the-dev-dir



## Docker

本Demo对Docker版本的要求是高于18.06

docker安装

到： https://download.docker.com/linux/static/stable/x86_64/ 下载需要版本的docker

此处用到了代理来加速下载，如果不用代理速度很慢。
```docker
使用代理：curl -x socks5://192.168.16.233:33333 -O https://download.docker.com/linux/static/stable/x86_64/docker-18.09.8.tgz
不用代理：curl -O https://download.docker.com/linux/static/stable/x86_64/docker-18.09.8.tgz
```

解压

```unzip
tar -xzvf docker-18.09.8.tgz
```

docker-compose 安装

到：https://github.com/docker/compose/releases/ 下载需要的版本的docker compose

```dockercompse
curl -L -O https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)

```

将下载下来的docker-compose文件更名为：docker-compose，然后赋予777 级别的权限

```mv
# mv docker-compose-Linux-x86_64 docker-compose
# chmod 777 docker-compose
```

将docker-compose文件和上面的docker二进制文件放在一起，并将整个目录放在一个妥善的位置，比如 /opt/local/docker 这样的目录结构

编辑下面的文件，文件名为docker.service, 注意这两行，要和上面的路径对应：

Environment="PATH=/root/bin/docker:/bin:/sbin:/usr/bin:/usr/sbin"
ExecStart=/root/bin/docker/dockerd --log-level=error

vi docker.service

```file
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.io

[Service]
Environment="PATH=/opt/local/docker:/bin:/sbin:/usr/bin:/usr/sbin"
ExecStart=/opt/local/docker/dockerd --log-level=error
ExecReload=/bin/kill -s HUP $MAINPID
Restart=on-failure
RestartSec=5
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target
```

**将该文件放到目录：/etc/systemd/system/ 下面**

```mv
mv docker.service /etc/systemd/system/
```

在 /etc/ 目录下增加 docker 目录，然后增加文件 /etc/docker/daemon.json

内容如下，并保存：

```aa
{
    "registry-mirrors": ["http://813c39a0.m.daocloud.io"],
    "max-concurrent-downloads": 20
}
```

增加环境变量

export PATH=/opt/local/docker:$PATH

然后生效

```aa
source /etc/profile
```

关闭selinux(重要)


执行下面命令关闭selinux

```d
setenforce 0
```

然后修改 /etc/sysconfig/selinux 文件中的 SELINUX=disabled，防止selinux在系统重启之后被打开

执行下面命令启动docker：

```ss
systemctl stop firewalld
systemctl disable firewalld
systemctl daemon-reload
systemctl start docker
```

检查docker 版本是否正确

```dockerversion
# docker version
Client: Docker Engine - Community
 Version:           18.09.8
 API version:       1.39
 Go version:        go1.10.8
 Git commit:        0dd43dd87f
 Built:             Wed Jul 17 17:38:58 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.8
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.8
  Git commit:       0dd43dd87f
  Built:            Wed Jul 17 17:48:49 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```

检查docker-compose 版本看看是否正确
```dockercompose
# docker-compose version
docker-compose version 1.21.2, build a133471
docker-py version: 3.3.0
CPython version: 3.6.5
OpenSSL version: OpenSSL 1.0.1t  3 May 2016
```

拉一个镜像看看docker加速器是否启用：

```dockerpull
docker pull mysql
```

如果看到镜像拉取速度较快，说明上面的docker加速器配置正确。至此，docker和docker compose安装完成

## Git 

系统自带的git版本太低，先卸载掉：
```removegit
yum remove git
```

然后到 https://mirrors.edge.kernel.org/pub/software/scm/git/ 找到合适的版本，此处选择了2.9.5版

```downloadgit
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.9.5.tar.gz
```

下载之后解压，进入解压之后的路径。

编译git之前，要升级系统，以及安装一系列依赖工具，执行如下命令：

```update
yum -y update
yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
```

上面的命令成功执行完之后，将git编译安装到 /opt/local/git 目录下

```gitbuild
make prefix=/opt/local/git all
make prefix=/opt/local/git install
```

可以看到git已经安装到/opt/local/git目录下，现在需要增加下面的环境变量(注意路径正确)：

```githong
# ========== git settings =====================
GIT_HOME=/opt/local/git/bin
PATH=$GIT_HOME:$PATH
export PATH
```

执行git version，看到正确的版本信息，说明安装成功
```newgit
# git version
git version 2.9.5
```

## Golang


需要到 https://golang.org/dl/

选择合适的版本进行下载，此处是LINUX故选择LINUX版本。

```aa
curl -O  https://dl.google.com/go/go1.10.linux-amd64.tar.gz
```

解压

```unzip
tar xzf go1.10.linux-amd64.tar.gz
```

将解压的目录移到合适的位置，比如 /opt/local/go

设置以下几个环境变量
```
# ========== Golang settings =====================
export GOROOT="/opt/local/go"
export GOPATH="/root/goprojects"
export GOBIN=$GOPATH/bin
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```
其中需要注意的是：

GOROOT是指golang编译器的位置

GOPATH是指项目根目录的位置, 使用go get拉下来的代码和编译出的二进制文件会放在这个目录里。

GOBIN是指项目目录中bin目录所在的位置


# 安装程序

## 拉取最新代码

此处重要需要解决的是Git的梯子问题，如果不设置梯子，可能会很慢，或者超时，此处的梯子需要自己的资源。

设置

```set
git config --global http.proxy 'http://192.168.16.233:33333'

git config --global https.proxy 'http://192.168.16.233:33333'

git config --global http.proxy 'socks5://192.168.16.233:33333'

git config --global https.proxy 'socks5://192.168.16.233:33333'
```

取消梯子

```unset
git config --global --unset http.proxy

git config --global --unset https.proxy
```

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


通过命令可以看到四个容器已经启动起来

```dockerpsa
[root@ethereum-192-168-16-60 basic-network]# docker ps -a
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                                            NAMES
e015028ac314        hyperledger/fabric-peer      "peer node start"        5 minutes ago       Up 5 minutes        0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
bdcb9cb18086        hyperledger/fabric-orderer   "orderer"                5 minutes ago       Up 5 minutes        0.0.0.0:7050->7050/tcp                           orderer.example.com
48f5341cc65a        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   5 minutes ago       Up 5 minutes        4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp       couchdb
922314e386e4        hyperledger/fabric-ca        "sh -c 'fabric-ca-se…"   5 minutes ago       Up 5 minutes        0.0.0.0:7054->7054/tcp                           ca.example.com

```

此时可以通过命令查看刚才创建的net_basic网络的状况

```networkinspect
$ docker network inspect net_basic

    {
        "Name": "net_basic",
        "Id": "62e9d37d00a0eda6c6301a76022c695f8e01258edaba6f65e876166164466ee5",
        "Created": "2018-11-07T13:46:30.4992927Z",
        "Containers": {
            "1fa1fd107bfbe61522e4a26a57c2178d82b2918d5d423e7ee626c79b8a233624": {
                "Name": "orderer.example.com",
                "IPv4Address": "172.20.0.4/16",
            },
            "469201085a20b6a8f476d1ac993abce3103e59e3a23b9125032b77b02b715f2c": {
                "Name": "ca.example.com",
                "IPv4Address": "172.20.0.2/16",
            },
            "53fe614274f7a40392210f980b53b421e242484dd3deac52bbfe49cb636ce720": {
                "Name": "couchdb",
                "IPv4Address": "172.20.0.3/16",
            },
            "ada3d078989b568c6e060fa7bf62301b4bf55bed8ac1c938d514c81c42d8727a": {
                "Name": "peer0.org1.example.com",
                "IPv4Address": "172.20.0.5/16",
            }
        },
        "Labels": {}
    }
```

从上面的网络地址和名字关系，可以看到四个容器的IPV4地址等相关信息。

# 测试

## 作为 MagnetoCorp 使用

本例子程序提供了用于监控状态的工具，利用该工具可以清楚的关注到docker容器的输出，这些输出信息中包含了各种有用的信息比如智能合约的安装执行等。

该工具位于：

fabric-samples/commercial-paper/organization/magnetocorp/configuration/cli/monitordocker.sh

执行下面的命令可以关注net_basic网络的运行情况：

```aaa

(magnetocorp admin)$ cd commercial-paper/organization/magnetocorp/configuration/cli/
(magnetocorp admin)$ ./monitordocker.sh net_basic
...
latest: Pulling from gliderlabs/logspout
4fe2ade4980c: Pull complete
decca452f519: Pull complete
(...)
Starting monitoring on all containers on the network net_basic
b7f3586e5d0233de5a454df369b8eadab0613886fc9877529587345fc01a3582

```

现在监测工具开始监测网络的输出了，我们打开另外一个网络终端，来做一些操作，观察其输出情况。

TD：首先拉起一个 MagnetoCorp的进程

```startMagenet
(magnetocorp admin)$ cd commercial-paper/organization/magnetocorp/configuration/cli/
(magnetocorp admin)$ docker-compose -f docker-compose.yml up -d cliMagnetoCorp

Pulling cliMagnetoCorp (hyperledger/fabric-tools:)...
latest: Pulling from hyperledger/fabric-tools
3b37166ec614: Already exists
(...)
Digest: sha256:058cff3b378c1f3ebe35d56deb7bf33171bf19b327d91b452991509b8e9c7870
Status: Downloaded newer image for hyperledger/fabric-tools:latest
Creating cliMagnetoCorp ... done
```

此时可以看到一个 hyperledger/fabric-tools 镜像所产生的容器已经被拉起

```dockerps
(magnetocorp admin)$ docker ps

CONTAINER ID        IMAGE                        COMMAND                  CREATED              STATUS              PORTS                                            NAMES
562a88b25149        hyperledger/fabric-tools     "/bin/bash"              About a minute ago   Up About a minute                                                    cliMagnetoCorp
b7f3586e5d02        gliderlabs/logspout          "/bin/logspout"          7 minutes ago        Up 7 minutes        127.0.0.1:8000->80/tcp                           logspout
ada3d078989b        hyperledger/fabric-peer      "peer node start"        29 minutes ago       Up 29 minutes       0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp   peer0.org1.example.com
1fa1fd107bfb        hyperledger/fabric-orderer   "orderer"                29 minutes ago       Up 29 minutes       0.0.0.0:7050->7050/tcp                           orderer.example.com
53fe614274f7        hyperledger/fabric-couchdb   "tini -- /docker-ent…"   29 minutes ago       Up 29 minutes       4369/tcp, 9100/tcp, 0.0.0.0:5984->5984/tcp       couchdb
469201085a20        hyperledger/fabric-ca        "sh -c 'fabric-ca-se…"   29 minutes ago       Up 29 minutes       0.0.0.0:7054->7054/tcp                           ca.example.com

```

## 智能合约 Smart contract

在 Commercial paper 这个例子中提到的三种行为：

- issue
- buy
- redeem

是PaperNet的三种智能合约方式的体现。TD：打开代码阅读智能合约

使用编辑器打开合约代码。

```viewcode
 commercial-paper/organization/magnetocorp/contract
```

在lib目录下可以查看 papercontract.js 文件。目前该版本的合约代码使用js编写。

Smart contracts are the focus of application development, and are contained within a Hyperledger Fabric artifact called chaincode. One or more smart contracts can be defined within a single chaincode, and installing a chaincode will allow them to be consumed by the different organizations in PaperNet. It means that only administrators need to worry about chaincode; everyone else can think in terms of smart contracts.

智能合约是区块链体系里实现业务逻辑的核心方式，而在hyperledger中，一个或多个只能合约组成一个chaincode(链代码)。

MagnetoCorp的管理员使用 peer chaincode install 命令，从本地机器拷贝 papercontract 智能合约到 peer的容器中，一旦智能合约被安装到 peer 节点并且在 channel 中被实例化，papercontract就可以被应用所调用。

TD： and interact with the ledger database via the putState() and getState() Fabric APIs. Examine how these APIs are used by StateList class within ledger-api\statelist.js.

可以查看 ledger-api/statelist.js 里面的源代码获得更多信息。

现在我们来以MagnetoCorp 管理员的身份安装 papercontract。在 MagnetoCorp 管理员的命令行窗口中，通过对cliMagnetCorp 容器执行 docker exec 命令来运行 peer chaincode install 指令，以此实现安装。

```installchaincode
(magnetocorp admin)$ docker exec cliMagnetoCorp peer chaincode install -n papercontract -v 0 -p /opt/gopath/src/github.com/contract -l node

2018-11-07 14:21:48.400 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2018-11-07 14:21:48.400 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2018-11-07 14:21:48.466 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" >
```

执行时，相应的，我们会在一直监控的屏幕上TD：此处增加锚点 看到如下输出：

```output

peer0.org1.example.com|2019-07-21 09:21:15.480 UTC [endorser] callChaincode -> INFO 037 [][7a4e9834] Entry chaincode: name:"lscc"
               couchdb|[notice] 2019-07-21T09:21:15.482738Z nonode@nohost <0.27044.32> 912847fcd0 couchdb:5984 172.19.0.5 undefined GET /mychannel_lscc 404 ok 0
peer0.org1.example.com|2019-07-21 09:21:15.488 UTC [couchdb] CreateDatabaseIfNotExist -> INFO 038 Created state database mychannel_lscc
               couchdb|[notice] 2019-07-21T09:21:15.488095Z nonode@nohost <0.27044.32> 8b5b32b64b couchdb:5984 172.19.0.5 undefined PUT /mychannel_lscc 201 ok 5
peer0.org1.example.com|2019-07-21 09:21:15.489 UTC [lscc] executeInstall -> INFO 039 Installed Chaincode [papercontract] Version [0] to peer
peer0.org1.example.com|2019-07-21 09:21:15.489 UTC [endorser] callChaincode -> INFO 03a [][7a4e9834] Exit chaincode: name:"lscc"  (9ms)
               couchdb|[notice] 2019-07-21T09:21:15.489032Z nonode@nohost <0.27044.32> ba8ff625cd couchdb:5984 172.19.0.5 undefined GET /mychannel_lscc/papercontract?attachments=true 404 ok 1
peer0.org1.example.com|2019-07-21 09:21:15.489 UTC [comm.grpc.server] 1 -> INFO 03b unary call completed {"grpc.start_time": "2019-07-21T09:21:15.479Z", "grpc.service": "protos.Endorser", "grpc.method": "ProcessProposal", "grpc.peer_address": "172.19.0.7:57948", "grpc.code": "OK", "grpc.call_duration": "9.314212ms"}

```

表明智能合约被正常的安装

Note how peer chaincode install command specified the smart contract path, -p, relative to the cliMagnetoCorp container’s file system: /opt/gopath/src/github.com/contract. This path has been mapped to the local file system path .../organization/magnetocorp/contract via the magnetocorp/configuration/cli/docker-compose.yml file:

注意命令中的 -p TD：-p不是绑定端口么？总之这里是是用了容器内部的文件，该文件其实是从本地映射上去的。映射关系如下：

```volumn
volumes:
    - ...
    - ./../../../../organization/magnetocorp:/opt/gopath/src/github.com/
    - ...
```

## 初始化智能合约

目前，papercontract的相关链代码已经成功的安装到了PaperNet的节点上，管理员可以在不同网络的通道上启用该合约代码，从而被连接到通道上的应用程序调用；但因为本例中只是为PaperNet启动了一个基本网络，所以此处只在一个单独的通道(mychannel)中启用papercontract。

通过下面的代码在mychannel中初始化 papercontract

```instantiate
(magnetocorp admin)$ docker exec cliMagnetoCorp peer chaincode instantiate -n papercontract -v 0 -l node -c '{"Args":["org.papernet.commercialpaper:instantiate"]}' -C mychannel -P "AND ('Org1MSP.member')"

2018-11-07 14:22:11.162 UTC [chaincodeCmd] InitCmdFactory -> INFO 001 Retrieved channel (mychannel) orderer endpoint: orderer.example.com:7050
2018-11-07 14:22:11.163 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default escc
2018-11-07 14:22:11.163 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default vscc
```

该命令的执行时间可能较长。TD：解释为什么

TD：执行该命令的时候，看到output那边有一大堆404，但是似乎又没问题。要看看是什么原因。

## Application dependencies

此处要注意npm-global的位置，我改动到 /opt/local目录下面了，参考的是这个文章：

https://stackoverflow.com/questions/29468404/gyp-warn-eacces-user-root-does-not-have-permission-to-access-the-dev-dir

## Wallet



