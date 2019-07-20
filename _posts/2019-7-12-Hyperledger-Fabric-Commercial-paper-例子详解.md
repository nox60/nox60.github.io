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


