```start.sh
#!/bin/bash

rm -rf /root/codes/temp/crypto-config
rm -rf /root/codes/temp/orderer_ledger
rm -rf /root/codes/temp/peer_ledger
rm -rf /root/codes/temp/channel.tx
rm -rf /root/codes/temp/genesis_block.pb

ps -ef | grep orderer | grep -v grep | awk '{print $2}' | xargs kill -9

cryptogen generate \
--config=./crypto-config.yaml \
--output="crypto-config"

# 创世区块
configtxgen -outputBlock genesis_block.pb \
-profile TwoOrgsOrdererGenesis \
-channelID orderer-system-channel

# tx
configtxgen -profile TwoOrgsChannel \
-outputCreateChannelTx /root/codes/temp/channel.tx \
-channelID mychannel

# start orderer
orderer &

# create channel
peer channel create -o orderer.test.com:7050 \
-c ca -f /root/codes/temp/channel.tx \
--tls true --cafile /root/codes/temp/crypto-config/ordererOrganizations/test.com/tlsca/tlsca.test.com-cert.pem
```