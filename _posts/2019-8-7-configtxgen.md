# configtxgen

The `configtxgen` command allows users to create and inspect channel config
related artifacts.  The content of the generated artifacts is dictated by the
contents of `configtx.yaml`.


`configtxgen` 工具能够让用户创建channel和查看channel对应的配置信息。相关的内容要参考`configtx.yaml` 配置文件。

## Syntax

The `configtxgen` tool has no sub-commands, but supports flags which can be set
to accomplish a number of tasks.

`configtxgen` 工具没有子命令，但是支持不同的参数已完成不同的任务。

## configtxgen
```
Usage of configtxgen:
  -asOrg string
    	Performs the config generation as a particular organization (by name), only including values in the write set that org (likely) has privilege to set

      为指名的组织生成配置文件，只包含 具有写集合的组织的？写集合？

  -channelCreateTxBaseProfile string
    	Specifies a profile to consider as the orderer system channel current state to allow modification of non-application parameters during channel create tx generation. Only valid in conjunction with 'outputCreateChannelTx'.
      
      指定一个profile文件
  -channelID string
    	The channel ID to use in the configtx
      通道ID
  -configPath string
    	The path containing the configuration to use (if set)
      配置文件地址，如果设置得有的话
  -inspectBlock string
    	Prints the configuration contained in the block at the specified path
  -inspectChannelCreateTx string
    	Prints the configuration contained in the transaction at the specified path
  -outputAnchorPeersUpdate string
    	Creates an config update to update an anchor peer (works only with the default channel creation, and only for the first update)
  -outputBlock string
    	The path to write the genesis block to (if set)
  -outputCreateChannelTx string
    	The path to write a channel creation configtx to (if set)
  -printOrg string
    	Prints the definition of an organization as JSON. (useful for adding an org to a channel manually)
  -profile string
    	The profile from configtx.yaml to use for generation. (default "SampleInsecureSolo")
  -version
    	Show version information
```

## Usage

### Output a genesis block

Write a genesis block to `genesis_block.pb` for channel `orderer-system-channel`
for profile `SampleSingleMSPSoloV1_1`.

为通道 `orderer-system-channel` 按照配置信息 `SampleSingleMSPSoloV1_1` 生成创世区块

```
configtxgen -outputBlock genesis_block.pb -profile SampleSingleMSPSoloV1_1 -channelID orderer-system-channel
```

要注意上面的配置信息段落：SampleSingleMSPSoloV1_1

### Output a channel creation tx

Write a channel creation transaction to `create_chan_tx.pb` for profile
`SampleSingleMSPChannelV1_1`.

按照配置信息 `SampleSingleMSPChannelV1_1`  生成通道的创造性交易？生成的文件名为 `create_chan_tx.pb`

```
configtxgen -outputCreateChannelTx create_chan_tx.pb -profile SampleSingleMSPChannelV1_1 -channelID application-channel-1
```

### Inspect a genesis block

Print the contents of a genesis block named `genesis_block.pb` to the screen as
JSON.

将一个名为`genesis_block.pb`的创世区块信息以为json的格式打印出来。

```
configtxgen -inspectBlock genesis_block.pb
```

### Inspect a channel creation tx

Print the contents of a channel creation tx named `create_chan_tx.pb` to the
screen as JSON.

将一个名为 `create_chan_tx.pb` 创建通道的交易以json格式打印出来。

```
configtxgen -inspectChannelCreateTx create_chan_tx.pb
```

### Print an organization definition

Construct an organization definition based on the parameters such as MSPDir
from `configtx.yaml` and print it as JSON to the screen. (This output is useful
for channel reconfiguration workflows, such as adding a member).



将一个组织的配置参数信息，比如 定义在 `configtx.yaml`中的 MSPDir 信息以json格式打印出来（这样的输出信息在重新配置通道的时候比较有用，比如增加一个成员？'peer?'） 

```
configtxgen -printOrg Org1
```

### Output anchor peer tx

Output a configuration update transaction to `anchor_peer_tx.pb` which sets the
anchor peers for organization Org1 as defined in profile
SampleSingleMSPChannelV1_1 based on `configtx.yaml`.

```
configtxgen -outputAnchorPeersUpdate anchor_peer_tx.pb -profile SampleSingleMSPChannelV1_1 -asOrg Org1
```

## Configuration

The `configtxgen` tool's output is largely controlled by the content of
`configtx.yaml`.  This file is searched for at `FABRIC_CFG_PATH` and must be
present for `configtxgen` to operate.

This configuration file may be edited, or, individual properties may be
overridden by setting environment variables, such as
`CONFIGTX_ORDERER_ORDERERTYPE=kafka`.


For many `configtxgen` operations, a profile name must be supplied.  Profiles
are a way to express multiple similar configurations in a single file.  For
instance, one profile might define a channel with 3 orgs, and another might
define one with 4 orgs.  To accomplish this without the length of the file
becoming burdensome, `configtx.yaml` depends on the standard YAML feature of
anchors and references.  Base parts of the configuration are tagged with an
anchor like `&OrdererDefaults` and then merged into a profile with a reference
like `<<: *OrdererDefaults`.  Note, when `configtxgen` is operating under a
profile, environment variable overrides do not need to include the profile
prefix and may be referenced relative to the root element of the profile.  For
instance, do not specify
`CONFIGTX_PROFILE_SAMPLEINSECURESOLO_ORDERER_ORDERERTYPE`,
instead simply omit the profile specifics and use the `CONFIGTX` prefix
followed by the elements relative to the profile name such as
`CONFIGTX_ORDERER_ORDERERTYPE`.

Refer to the sample `configtx.yaml` shipped with Fabric for all possible
configuration options.  You may find this file in the `config` directory of
the release artifacts tar, or you may find it under the `sampleconfig` folder
if you are building from source.


