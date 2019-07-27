# 总启动脚本

```startsh

set -ev

# don't rewrite paths for Windows Git Bash users
export MSYS_NO_PATHCONV=1

docker-compose -f docker-compose.yml down

docker-compose -f docker-compose.yml up -d ca.example.com orderer.example.com peer0.org1.example.com couchdb
# 此处启动四个容器：
CA容器
orderer容器
peer节点容器
couchdb容器

docker ps -a

# wait for Hyperledger Fabric to start
# incase of errors when running later commands, issue export FABRIC_START_TIMEOUT=<larger number>
export FABRIC_START_TIMEOUT=10
#echo ${FABRIC_START_TIMEOUT}
sleep ${FABRIC_START_TIMEOUT}

# Create the channel
# 利用上面创建的  peer0.org1.example.com 节点创建一个channel，执行命令：peer channel create -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx
# 命令解读：
# 

docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel create -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx

# Join peer0.org1.example.com to the channel.
docker exec -e "CORE_PEER_LOCALMSPID=Org1MSP" -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com peer channel join -b mychannel.block

```


# CA容器

容器所用镜像：hyperledger/fabric-ca

```ca
    ca.example.com:
      image: hyperledger/fabric-ca
      environment:
        - FABRIC_CA_HOME=/etc/hyperledger/fabric-ca-server
        - FABRIC_CA_SERVER_CA_NAME=ca.example.com
        - FABRIC_CA_SERVER_CA_CERTFILE=/etc/hyperledger/fabric-ca-server-config/ca.org1.example.com-cert.pem
        - FABRIC_CA_SERVER_CA_KEYFILE=/etc/hyperledger/fabric-ca-server-config/4239aa0dcd76daeeb8ba0cda701851d14504d31aad1b2ddddbac6a57365e497c_sk
        # FABRIC_CA_SERVER_CA_CERTFILE: 公钥
        # FABRIC_CA_SERVER_CA_KEYFILE : 私钥
      
      ports:
        - "7054:7054"
      command: sh -c 'fabric-ca-server start -b admin:adminpw'
        # 这行命令是这个容器启动时调用的命令，表示创建一个ca并且初始化管理员账户和密码，在之前的文章中有介绍，参见对CA的详解：https://nox60.github.io/2019/07/18/Hyperledge-fabirc-ca/
      volumes:
        - ./crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config
        # 此处就是把本地的一些准备好的ca信息挂在到容器中去
      container_name: ca.example.com
      networks:
        - basic
```

用二进制程序的方式模拟上面的docker

首先，要注意这行挂载，上面容器运行的时候，有些文件是从外面挂载进去的：

./crypto-config/peerOrganizations/org1.example.com/ca/:/etc/hyperledger/fabric-ca-server-config

熟悉Docker的话，对这行应该一目了然，这是把本地目录 ./crypto-config/peerOrganizations/org1.example.com/ca/ 向 容器里的 /etc/hyperledger/fabric-ca-server-config 目录映射

fabric-ca-server的编译方式（链接）

注意上面的第一个环境变量：FABRIC_CA_HOME
这个环境变量是CA服务器的home目录。
- 如果 -home 选项有配置，则使用其配置的值
- 否则，如果 FABRIC_CA_SERVER_HOME 环境变量值有设置，则使用其值
- 否则，如果 FABRIC_CA_HOME 环境变量值有设置，则使用其值
- 否则，如果 CA_CFG_PATH 环境变量值有值，则使用其值
- 如果以上都没有，则使用当前工作目录。

上面第二个环境变量：FABRIC_CA_SERVER_CA_NAME
对应的命令行中的 --ca.name 参数，指定 ca服务器的名称

第三个环境变量：FABRIC_CA_SERVER_CA_CERTFILE
指定服务器的公钥文件 TD：此处要分析研究该文件的来源, --ca.certfile

第四个环境变量：FABRIC_CA_SERVER_CA_KEYFILE
指定服务器的私钥文件，--ca.keyfile

fabirc-ca-server相关密钥文件的生成文档参见(链接)

综上所述，模拟该docker启动的二进制命令是：

```d
fabric-ca-server start \
-b admin:adminpw \
-H /root/ca-server2/ \
--port 7088 \
--ca.name ca.example.com \
--ca.certfile /root/softwares/fabric-samples/basic-network/crypto-config/peerOrganizations/org1.example.com/ca/ca.org1.example.com-cert.pem \
--ca.keyfile /root/softwares/fabric-samples/basic-network/crypto-config/peerOrganizations/org1.example.com/ca/4239aa0dcd76daeeb8ba0cda701851d14504d31aad1b2ddddbac6a57365e497c_sk
```



# orderer容器

容器所用镜像：hyperledger/fabric-orderer

```eee
  orderer.example.com:
    container_name: orderer.example.com
    image: hyperledger/fabric-orderer
    environment:
      - FABRIC_LOGGING_SPEC=info
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/etc/hyperledger/configtx/genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/etc/hyperledger/msp/orderer/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/orderer
    command: orderer
    # 启动之后执行orderer命令
    ports:
      - 7050:7050
    volumes:
        - ./config/:/etc/hyperledger/configtx
        - ./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/:/etc/hyperledger/msp/orderer
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/:/etc/hyperledger/msp/peerOrg1
    networks:
      - basic

```

用二进制程序的方式模拟上面的docker

首先要注意的是三个挂载点：

1. 第一个挂载了 /config/ 目录，该目录里有orderer.yaml 配置文件；

2. 第二个挂载点挂载了对应的msp目录，里面包含了：。

3. 第三个挂载点对应了peer 的msp的CA 证书

上面的docker方式使用了五个环境变量对服务进行配置，而在二进制方式中，更多的配置方式来自于配置文件 orderer.yaml 里面的配置。



# peer节点容器

容器所用镜像：hyperledger/fabric-peer

```eee
  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    image: hyperledger/fabric-peer
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_PEER_ID=peer0.org1.example.com
      - FABRIC_LOGGING_SPEC=info
      - CORE_CHAINCODE_LOGGING_LEVEL=info
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/peer/
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      # # the following setting starts chaincode containers on the same
      # # bridge network as the peers
      # # https://docs.docker.com/compose/networking/
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_basic
      - CORE_LEDGER_STATE_STATEDATABASE=CouchDB
      - CORE_LEDGER_STATE_COUCHDBCONFIG_COUCHDBADDRESS=couchdb:5984
      # The CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME and CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD
      # provide the credentials for ledger to connect to CouchDB.  The username and password must
      # match the username and password set for the associated CouchDB.
      - CORE_LEDGER_STATE_COUCHDBCONFIG_USERNAME=
      - CORE_LEDGER_STATE_COUCHDBCONFIG_PASSWORD=
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: peer node start
    # command: peer node start --peer-chaincodedev=true
    ports:
      - 7051:7051
      - 7053:7053
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/msp/peer
        - ./crypto-config/peerOrganizations/org1.example.com/users:/etc/hyperledger/msp/users
        - ./config:/etc/hyperledger/configtx
    depends_on:
      - orderer.example.com
      - couchdb
    networks:
      - basic

```


# couchdb容器

容器所用镜像：hyperledger/fabric-couchdb

```eee
  couchdb:
    container_name: couchdb
    image: hyperledger/fabric-couchdb
    # Populate the COUCHDB_USER and COUCHDB_PASSWORD to set an admin user and password
    # for CouchDB.  This will prevent CouchDB from operating in an "Admin Party" mode.
    environment:
      - COUCHDB_USER=
      - COUCHDB_PASSWORD=
    ports:
      - 5984:5984
    networks:
      - basic

```

此处启动的couchdb是最简单的，没有进行太多配置，要查看其它程序怎么关联这个couchdb容器的


# cli容器？


```eee
  cli:
    container_name: cli
    image: hyperledger/fabric-tools
    tty: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - FABRIC_LOGGING_SPEC=info
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
      - CORE_CHAINCODE_KEEPALIVE=10
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - ./../chaincode/:/opt/gopath/src/github.com/
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
    networks:
        - basic
    #depends_on:
    #  - orderer.example.com
    #  - peer0.org1.example.com
    #  - couchdb

```