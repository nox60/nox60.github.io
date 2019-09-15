```start.sh

#!/bin/bash

rm -rf /root/codes/temp/crypto-config
rm -rf /root/codes/temp/orderer_ledger
rm -rf /root/codes/temp/peer_ledger
rm -rf /root/codes/temp/channel.tx
rm -rf /root/codes/temp/genesis_block.pb

ps -ef | grep orderer | grep -v grep | awk '{print $2}' | xargs kill -9
ps -ef | grep peer | grep -v grep | awk '{print $2}' | xargs kill -9

cryptogen generate \
--config=./crypto-config.yaml \
--output="crypto-config"

echo 'Create genesis block'
# 创世区块
configtxgen -outputBlock genesis_block.pb \
-profile TwoOrgsOrdererGenesis

echo 'Create tx'
# tx
configtxgen -profile TwoOrgsChannel \
-outputCreateChannelTx /root/codes/temp/channel.tx \
-channelID mychannel

echo 'start orderer'
# start orderer
nohup orderer > orderer.log   2>&1 &

sleep 2
echo 'start peer'
nohup peer node start > peer.log 2>&1 &

sleep 2
echo 'create channel'
# create channel
peer channel create -o orderer.test.com:7050 \
-c mychannel \
-f /root/codes/temp/channel.tx \
--tls true \
--cafile /root/codes/temp/crypto-config/ordererOrganizations/test.com/tlsca/tlsca.test.com-cert.pem


# 加入通道
peer channel join \
-b mychannel.block \
-o orderer.test.com:7050 \
--cafile /root/codes/temp/crypto-config/ordererOrganizations/test.com/tlsca/tlsca.test.com-cert.pem


# 列出所有的通道

peer channel list \
-o orderer.test.com:7050 \
--cafile /root/codes/temp/crypto-config/ordererOrganizations/test.com/tlsca/tlsca.test.com-cert.pem


peer channel getinfo \
-c mychannel \
-o orderer.test.com:7050 \
--tls true \
--cafile /root/codes/temp/crypto-config/ordererOrganizations/test.com/tlsca/tlsca.test.com-cert.pem
```

配置文件模板

https://github.com/hyperledger/fabric/blob/release-1.4/sampleconfig/configtx.yaml


byfn官方例子

```byfnsh

+ cryptogen generate --config=./crypto-config.yaml
org1.example.com
org2.example.com
+ res=0
+ set +x

Generate CCP files for Org1 and Org2
/root/codes/fabric-samples/bin/configtxgen
##########################################################
#########  Generating Orderer Genesis block ##############
##########################################################
CONSENSUS_TYPE=solo
+ '[' solo == solo ']'
+ configtxgen -profile TwoOrgsOrdererGenesis -channelID byfn-sys-channel -outputBlock ./channel-artifacts/genesis.block
2019-09-02 11:47:42.077 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-09-02 11:47:42.133 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002 orderer type: solo
2019-09-02 11:47:42.133 CST [common.tools.configtxgen.localconfig] Load -> INFO 003 Loaded configuration: /root/codes/fabric-samples/first-network/configtx.yaml
2019-09-02 11:47:42.189 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 004 orderer type: solo
2019-09-02 11:47:42.189 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 005 Loaded configuration: /root/codes/fabric-samples/first-network/configtx.yaml
2019-09-02 11:47:42.191 CST [common.tools.configtxgen] doOutputBlock -> INFO 006 Generating genesis block
2019-09-02 11:47:42.191 CST [common.tools.configtxgen] doOutputBlock -> INFO 007 Writing genesis block
+ res=0
+ set +x

#################################################################
### Generating channel configuration transaction 'channel.tx' ###
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
2019-09-02 11:47:42.215 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-09-02 11:47:42.272 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /root/codes/fabric-samples/first-network/configtx.yaml
2019-09-02 11:47:42.328 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
2019-09-02 11:47:42.328 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /root/codes/fabric-samples/first-network/configtx.yaml
2019-09-02 11:47:42.328 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 005 Generating new channel configtx
2019-09-02 11:47:42.330 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 006 Writing new channel tx
+ res=0
+ set +x

#################################################################
#######    Generating anchor peer update for Org1MSP   ##########
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
2019-09-02 11:47:42.354 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-09-02 11:47:42.410 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /root/codes/fabric-samples/first-network/configtx.yaml
2019-09-02 11:47:42.467 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
2019-09-02 11:47:42.467 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /root/codes/fabric-samples/first-network/configtx.yaml
2019-09-02 11:47:42.467 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 005 Generating anchor peer update
2019-09-02 11:47:42.468 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 006 Writing anchor peer update
+ res=0
+ set +x

#################################################################
#######    Generating anchor peer update for Org2MSP   ##########
#################################################################
+ configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP
2019-09-02 11:47:42.491 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2019-09-02 11:47:42.549 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /root/codes/fabric-samples/first-network/configtx.yaml
2019-09-02 11:47:42.606 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 orderer type: solo
2019-09-02 11:47:42.606 CST [common.tools.configtxgen.localconfig] LoadTopLevel -> INFO 004 Loaded configuration: /root/codes/fabric-samples/first-network/configtx.yaml
2019-09-02 11:47:42.606 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 005 Generating anchor peer update
2019-09-02 11:47:42.606 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 006 Writing anchor peer update
+ res=0
+ set +x

Creating network "net_byfn" with the default driver
Creating volume "net_orderer.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_peer1.org1.example.com" with default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating volume "net_peer1.org2.example.com" with default driver
Creating peer0.org1.example.com ... done
Creating peer1.org2.example.com ... done
Creating orderer.example.com    ... done
Creating peer0.org2.example.com ... done
Creating peer1.org1.example.com ... done
Creating cli                    ... done
CONTAINER ID        IMAGE                               COMMAND             CREATED                  STATUS                  PORTS                      NAMES
089a01b7d8e8        hyperledger/fabric-tools:latest     "/bin/bash"         Less than a second ago   Up Less than a second                              cli
bea694c61eab        hyperledger/fabric-peer:latest      "peer node start"   1 second ago             Up Less than a second   0.0.0.0:8051->8051/tcp     peer1.org1.example.com
7d65e0a2e138        hyperledger/fabric-peer:latest      "peer node start"   1 second ago             Up Less than a second   0.0.0.0:10051->10051/tcp   peer1.org2.example.com
6a738e12e77c        hyperledger/fabric-orderer:latest   "orderer"           1 second ago             Up Less than a second   0.0.0.0:7050->7050/tcp     orderer.example.com
15976701bf05        hyperledger/fabric-peer:latest      "peer node start"   1 second ago             Up Less than a second   0.0.0.0:9051->9051/tcp     peer0.org2.example.com
187045d6061a        hyperledger/fabric-peer:latest      "peer node start"   1 second ago             Up Less than a second   0.0.0.0:7051->7051/tcp     peer0.org1.example.com

 ____    _____      _      ____    _____
/ ___|  |_   _|    / \    |  _ \  |_   _|
\___ \    | |     / _ \   | |_) |   | |
 ___) |   | |    / ___ \  |  _ <    | |
|____/    |_|   /_/   \_\ |_| \_\   |_|

Build your first network (BYFN) end-to-end test

Channel name : mychannel
Creating channel...
+ peer channel create -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
+ res=0
+ set +x
2019-09-02 03:47:44.760 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2019-09-02 03:47:44.778 UTC [cli.common] readBlock -> INFO 002 Received block: 0
+ peer channel join -b mychannel.block
===================== Channel 'mychannel' created =====================

Having all peers join the channel...
+ res=0
+ set +x
2019-09-02 03:47:44.822 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2019-09-02 03:47:44.840 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
===================== peer0.org1 joined channel 'mychannel' =====================

+ peer channel join -b mychannel.block
+ res=0
+ set +x
2019-09-02 03:47:47.887 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2019-09-02 03:47:47.906 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
===================== peer1.org1 joined channel 'mychannel' =====================

+ peer channel join -b mychannel.block
+ res=0
+ set +x
2019-09-02 03:47:50.952 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2019-09-02 03:47:50.971 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
===================== peer0.org2 joined channel 'mychannel' =====================

+ peer channel join -b mychannel.block
+ res=0
+ set +x
2019-09-02 03:47:54.017 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2019-09-02 03:47:54.034 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
===================== peer1.org2 joined channel 'mychannel' =====================

Updating anchor peers for org1...
+ peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org1MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
+ res=0
+ set +x
2019-09-02 03:47:57.079 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2019-09-02 03:47:57.091 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
===================== Anchor peers updated for org 'Org1MSP' on channel 'mychannel' =====================

Updating anchor peers for org2...
+ peer channel update -o orderer.example.com:7050 -c mychannel -f ./channel-artifacts/Org2MSPanchors.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
+ res=0
+ set +x
2019-09-02 03:48:00.143 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2019-09-02 03:48:00.154 UTC [channelCmd] update -> INFO 002 Successfully submitted channel update
===================== Anchor peers updated for org 'Org2MSP' on channel 'mychannel' =====================

Installing chaincode on peer0.org1...
+ peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/
+ res=0
+ set +x
2019-09-02 03:48:03.213 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2019-09-02 03:48:03.213 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2019-09-02 03:48:03.353 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" >
===================== Chaincode is installed on peer0.org1 =====================

Install chaincode on peer0.org2...
+ peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/
+ res=0
+ set +x
2019-09-02 03:48:03.399 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2019-09-02 03:48:03.399 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2019-09-02 03:48:03.533 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" >
===================== Chaincode is installed on peer0.org2 =====================

Instantiating chaincode on peer0.org2...
+ peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc -l golang -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P 'AND ('\''Org1MSP.peer'\'','\''Org2MSP.peer'\'')'
+ res=0
+ set +x
2019-09-02 03:48:03.581 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2019-09-02 03:48:03.581 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
===================== Chaincode is instantiated on peer0.org2 on channel 'mychannel' =====================

Querying chaincode on peer0.org1...
===================== Querying on peer0.org1 on channel 'mychannel'... =====================
Attempting to Query peer0.org1 ...3 secs
+ peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
+ res=0
+ set +x

100
===================== Query successful on peer0.org1 on channel 'mychannel' =====================
Sending invoke transaction on peer0.org1 peer0.org2...
+ peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'
+ res=0
+ set +x
+ peer chaincode install -n mycc -v 1.0 -l golang -p github.com/chaincode/chaincode_example02/go/
2019-09-02 03:48:28.862 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200
===================== Invoke transaction successful on peer0.org1 peer0.org2 on channel 'mychannel' =====================

Installing chaincode on peer1.org2...
+ res=0
+ set +x
2019-09-02 03:48:28.911 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2019-09-02 03:48:28.911 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2019-09-02 03:48:29.049 UTC [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" >
===================== Chaincode is installed on peer1.org2 =====================

Querying chaincode on peer1.org2...
===================== Querying on peer1.org2 on channel 'mychannel'... =====================
Attempting to Query peer1.org2 ...3 secs
+ peer chaincode query -C mychannel -n mycc -c '{"Args":["query","a"]}'
+ res=0
+ set +x

```

```startprocess
启动流程

先启动orderer容器
  orderer.example.com:
    extends:
      file:   base/docker-compose-base.yaml
      service: orderer.example.com
    container_name: orderer.example.com
    networks:
      - byfn

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    extends:
      file:  base/docker-compose-base.yaml
      service: peer0.org1.example.com
    networks:
      - byfn

再启动peer0 org1

Creating volume "net_orderer.example.com" with default driver
Creating volume "net_peer0.org1.example.com" with default driver
Creating volume "net_peer1.org1.example.com" with default driver
Creating volume "net_peer0.org2.example.com" with default driver
Creating volume "net_peer1.org2.example.com" with default driver

```
