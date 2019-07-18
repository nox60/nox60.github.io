# Git环境配置

Git的下载，安装参见官网。

此处重要需要解决的是Git的梯子问题，因为后面编译Hyperledge-ca的时候，Go是需要在github上获取一些依赖代码的，如果不设置梯子，可能会很慢，或者超时，此处的梯子需要自己的资源。

设置

```set
git config --global http.proxy 'http://127.0.0.1:1080'

git config --global https.proxy 'http://127.0.0.1:1080'

git config --global http.proxy 'socks5://127.0.0.1:1080'

git config --global https.proxy 'socks5://127.0.0.1:1080'
```

取消

```unset
git config --global --unset http.proxy

git config --global --unset https.proxy
```

# Golang环境配置

需要到 https://golang.org/dl/

选择合适的版本进行下载，此处是LINUX故选择LINUX版本。

```aa
wget https://dl.google.com/go/go1.10.linux-amd64.tar.gz
```

解压

```unzip
tar xzf go1.10.linux-amd64.tar.gz
```

设置以下几个环境变量
```
# ========== Golang settings =====================
export GOROOT="/root/bin/go"
export GOPATH="/root/goprojects"
export GOBIN=$GOPATH/bin
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```
其中，

GOROOT是指golang编译器的位置

GOPATH是指项目根目录的位置, 使用go get拉下来的代码和编译出的二进制文件会放在这个目录里。

GOBIN是指项目目录中bin目录所在的位置

# Hyberledger server

```go
go get -u github.com/hyperledger/fabric-ca/cmd/...
```
执行之后，如果一切正常，GOPATH里面将会有拉下来的代码和编译之后的二进制文件。


## 原生方式启动 fabric-ca-server

首先需要编译程序

```go
go get -u github.com/hyperledger/fabric-ca/cmd/...
```
执行之后，如果一切正常，GOPATH里面将会有拉下来的代码和编译之后的二进制文件。



## 参数说明

| 缩写       |  参数     | 参类型  | 说明      | 例子|
| ---------  | --------- | ----   | -------   | --- |
|      | -&zwnj;-address      | string    | fabric-ca-server 监听地址 (默认) "0.0.0.0")  | <a name="锚点名称">例子</a>
|
| -b     | -&zwnj;-boot       | string    | The user:pass for bootstrap admin which is required to build default config file  | <a name="start_ca_server">例子</a> |
|  | -&zwnj;-ca.certfile | string    | PEM-encoded CA certificate file (default "ca-cert.pem")  | <a name="锚点名称">例子</a> |

| -H | -&zwnj;-home | string    | 服务端home目录，默认当前目录 | <a name="锚点名称">例子</a> | 


<a name="start_ca_server">用以下的命令启动fabirc-ca-server </a>

```start
fabric-ca-server start -b admin:adminpw
```

## Docker方式启动 fabric-ca-server

to do

# Hyberledger client



### 所属三级目录

## 另外一个二级目录