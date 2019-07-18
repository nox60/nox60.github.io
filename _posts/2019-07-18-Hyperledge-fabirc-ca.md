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

GOPATH是指项目根目录的位置,一般来说,这个变量会随着我们的项目改变而改变

GOBIN是指项目目录中bin目录所在的位置

# Hyberledger server

```go
go get -u github.com/hyperledger/fabric-ca/cmd/...
```

