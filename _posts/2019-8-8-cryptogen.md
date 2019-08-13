# cryptogen

`cryptogen` is an utility for generating Hyperledger Fabric key material.
It is provided as a means of preconfiguring a network for testing purposes.
It would normally not be used in the operation of a production network.

`cryptogen` 是一个生成Hyperledger Fabric相关秘钥的工具。为网络的预先配置提供了实现。一般来说不会用于生产环境。在生成创世区块之前，要用该工具先生成各种需要的秘钥。

## Syntax
## 语法

The ``cryptogen`` command has five subcommands, as follows:
`cryptogen` 命令有个五个子命令，如下：

  * help
  * generate
  * showtemplate
  * extend
  * version

## cryptogen help
```
usage: cryptogen [<flags>] <command> [<args> ...]

Utility for generating Hyperledger Fabric key material

Flags:
  --help  Show context-sensitive help (also try --help-long and --help-man).

Commands:
  help [<command>...]
    Show help.

  generate [<flags>]
    Generate key material

  showtemplate
    Show the default configuration template

  version
    Show version information

  extend [<flags>]
    Extend existing network
```


## cryptogen generate
```
usage: cryptogen generate [<flags>]

Generate key material

Flags:
  --help                    Show context-sensitive help (also try --help-long
                            and --help-man).
  --output="crypto-config"  The output directory in which to place artifacts
  --config=CONFIG           The configuration template to use
```


## cryptogen showtemplate
```
usage: cryptogen showtemplate

Show the default configuration template

Flags:
  --help  Show context-sensitive help (also try --help-long and --help-man).
```


## cryptogen extend
```
usage: cryptogen extend [<flags>]

Extend existing network

Flags:
  --help                   Show context-sensitive help (also try --help-long and
                           --help-man).
  --input="crypto-config"  The input directory in which existing network place
  --config=CONFIG          The configuration template to use
```


## cryptogen version
```
usage: cryptogen version

Show version information

Flags:
  --help  Show context-sensitive help (also try --help-long and --help-man).
```

## Usage

Here's an example using the different available flags on the ``cryptogen extend``
command.

```
    cryptogen extend --input="crypto-config" --config=config.yaml

    org3.example.com
```

Where config.yaml adds a new peer organization called ``org3.example.com``

## config.yaml 配置文件解析

```template


# ---------------------------------------------------------------------------
# "OrdererOrgs" - Definition of organizations managing orderer nodes 
# 定义管理排序节点的相关组织信息
# ---------------------------------------------------------------------------
OrdererOrgs:
  # ---------------------------------------------------------------------------
  # Orderer 
  # 节点名称和域名
  # ---------------------------------------------------------------------------
  - Name: Orderer
    Domain: example.com

    # ---------------------------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
    # ---------------------------------------------------------------------------
    Specs:
      - Hostname: orderer

# ---------------------------------------------------------------------------
# "PeerOrgs" - Definition of organizations managing peer nodes 定义管理peer的组织名称
# ---------------------------------------------------------------------------
PeerOrgs:
  # ---------------------------------------------------------------------------
  # Org1
  # ---------------------------------------------------------------------------
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: false

    # ---------------------------------------------------------------------------
    # "CA"
    # ---------------------------------------------------------------------------
    # Uncomment this section to enable the explicit definition of the CA for this
    # organization.  This entry is a Spec.  See "Specs" section below for details.
    # 该组织的CA信息？CSR信息？
    # ---------------------------------------------------------------------------
    # CA:
    #    Hostname: ca # implicitly ca.org1.example.com
    #    Country: US
    #    Province: California
    #    Locality: San Francisco
    #    OrganizationalUnit: Hyperledger Fabric
    #    StreetAddress: address for org # default nil
    #    PostalCode: postalCode for org # default nil

    # ---------------------------------------------------------------------------
    # "Specs"
    # ---------------------------------------------------------------------------
    # Uncomment this section to enable the explicit definition of hosts in your
    # configuration.  Most users will want to use Template, below
    #
    # Specs is an array of Spec entries.  Each Spec entry consists of two fields:
    #   - Hostname:   (Required) The desired hostname, sans the domain.
    #                 必填，隐式的会生成："foo.org1.example.com" ？这个需要验证
    #   - CommonName: (Optional) Specifies the template or explicit override for
    #                 the CN.  By default, this is the template:
    #
    #                              "{{.Hostname}}.{{.Domain}}"
    #
    #                 which obtains its values from the Spec.Hostname and
    #                 Org.Domain, respectively.
    #   - SANS:       (Optional) Specifies one or more Subject Alternative Names
    #                 to be set in the resulting x509. Accepts template
    #                 variables {{.Hostname}}, {{.Domain}}, {{.CommonName}}. IP
    #                 addresses provided here will be properly recognized. Other
    #                 values will be taken as DNS names.
    #                 NOTE: Two implicit entries are created for you:
    #                     - {{ .CommonName }}
    #                     - {{ .Hostname }}
    # ---------------------------------------------------------------------------
    # Specs:
    #   - Hostname: foo # implicitly "foo.org1.example.com"
    #     CommonName: foo27.org5.example.com # overrides Hostname-based FQDN set above
    #     SANS:
    #       - "bar.{{.Domain}}"
    #       - "altfoo.{{.Domain}}"
    #       - "{{.Hostname}}.org6.net"
    #       - 172.16.10.31
    #   - Hostname: bar
    #   - Hostname: baz

    # ---------------------------------------------------------------------------
    # "Template"
    # ---------------------------------------------------------------------------
    # Allows for the definition of 1 or more hosts that are created sequentially
    # from a template. By default, this looks like "peer%d" from 0 to Count-1.
    # You may override the number of nodes (Count), the starting index (Start)
    # or the template used to construct the name (Hostname).
    #
    # Note: Template and Specs are not mutually exclusive.  You may define both
    # sections and the aggregate nodes will be created for you.  Take care with
    # name collisions
    # ---------------------------------------------------------------------------
    Template:
      Count: 1
      # Start: 5
      # Hostname: {{.Prefix}}{{.Index}} # default
      # SANS:
      #   - "{{.Hostname}}.alt.{{.Domain}}"

    # ---------------------------------------------------------------------------
    # "Users"
    # ---------------------------------------------------------------------------
    # Count: The number of user accounts _in addition_ to Admin
    # ---------------------------------------------------------------------------
    Users:
      Count: 1

  # ---------------------------------------------------------------------------
  # Org2: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: Org2
    Domain: org2.example.com
    EnableNodeOUs: false
    Template:
      Count: 1
    Users:
      Count: 1
```

## 测试执行

执行命令：

```testcommand
cryptogen generate \
--config=./crypto-config.yaml \
--output="crypto-config-test" 
```

对应的配置文件 crypto-config.yaml如：


```aaa

OrdererOrgs:
  - Name: Orderer
    Domain: mydomain.net
    Specs:
      - Hostname: orderer
PeerOrgs:
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true
    Template:
      Count: 2
    Users:
      Count: 2
```

会生成如下文件，目录结构如下，可以看到，生成了以下文件：

- ca的公私钥
- org的msp目录
- orderer节点的msp目录
- admin的msp目录

```tree
.
└── mydomain.net
    ├── ca
    │   ├── 547f11276aa5a436d1d636b7c96507fbb42f4b31452a4aa2eb4697149bb909ea_sk
    │   └── ca.mydomain.net-cert.pem
    ├── msp
    │   ├── admincerts
    │   │   └── Admin@mydomain.net-cert.pem
    │   ├── cacerts
    │   │   └── ca.mydomain.net-cert.pem
    │   └── tlscacerts
    │       └── tlsca.mydomain.net-cert.pem
    ├── orderers
    │   └── orderer.mydomain.net
    │       ├── msp
    │       │   ├── admincerts
    │       │   │   └── Admin@mydomain.net-cert.pem
    │       │   ├── cacerts
    │       │   │   └── ca.mydomain.net-cert.pem
    │       │   ├── keystore
    │       │   │   └── 1497ce9d6db52da30d589e25501b7b4e3cb96568e3385fc4150dcdf384cd705b_sk
    │       │   ├── signcerts
    │       │   │   └── orderer.mydomain.net-cert.pem
    │       │   └── tlscacerts
    │       │       └── tlsca.mydomain.net-cert.pem
    │       └── tls
    │           ├── ca.crt
    │           ├── server.crt
    │           └── server.key
    ├── tlsca
    │   ├── b9ae9473c90b0721a10da8f35ce7bf267fceab671b1d89ec2f81b3eae701ba2a_sk
    │   └── tlsca.mydomain.net-cert.pem
    └── users
        └── Admin@mydomain.net
            ├── msp
            │   ├── admincerts
            │   │   └── Admin@mydomain.net-cert.pem
            │   ├── cacerts
            │   │   └── ca.mydomain.net-cert.pem
            │   ├── keystore
            │   │   └── 00483868486b00f776ebe38c4df8da91fa5627be9ac1201a496a012b8e5239e6_sk
            │   ├── signcerts
            │   │   └── Admin@mydomain.net-cert.pem
            │   └── tlscacerts
            │       └── tlsca.mydomain.net-cert.pem
            └── tls
                ├── ca.crt
                ├── client.crt
                └── client.key
```



用上面的信息生成创世区块，生成创世区块要用到的配置文件是`configtx.yaml`：

注意命令里面的相关信息


```genen
configtxgen -profile TwoOrgsOrdererGenesis -channelID my-channel -outputBlock ./channel-artifacts/genesis.block
configtxgen -profile OnlyOrdererGenesis -channelID myme-channel -outputBlock ./channel-artifacts/genesis.block

```

- 其中，-profile后面跟的是 configtx.yaml 配置文件中，最后的profile块里面定义的profile名称，用这样的方式，可以简化配置文件的编写。
- channelID 需要明确
- -outputBlock 就是输出的创世区块位置


## 利用创世区块拉起orderer服务

注意上面生成的目录中

```pt
.
└── mydomain.net
    │   
    ├── msp
    ├── orderers
    │   └── orderer.mydomain.net
    │       ├── msp
```

/root/softwares/fabric-samples/first-network/channel-artifacts

/root/softwares/fabric-samples/first-network/crypto-config-test/ordererOrganizations/mydomain.net/orderers/orderer.mydomain.net/msp

参考下面的orderer.yaml配置文件：

```orderer

```


出错信息：

2019-08-14 00:30:51.446 CST [orderer.commmon.multichannel] checkResourcesOrPanic -> PANI 004 [channel my-channel] config requires unsupported orderer capabilities: Orderer capability V1_4_2 is required but not supported: Orderer capability V1_4_2 is required but not supported
panic: [channel my-channel] config requires unsupported orderer capabilities: Orderer capability V1_4_2 is required but not supported: Orderer capability V1_4_2 is required but not supported



