# cryptogen

`cryptogen` is an utility for generating Hyperledger Fabric key material.
It is provided as a means of preconfiguring a network for testing purposes.
It would normally not be used in the operation of a production network.

`cryptogen` 是一个生成Hyperledger Fabric相关秘钥的工具。为网络的预先配置提供了实现。一般来说不会用于生产环境。

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