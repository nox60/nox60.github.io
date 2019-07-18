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
|      | -&zwnj;-address      | string    | fabric-ca-server 监听地址 (默认) "0.0.0.0")  | <a name="锚点名称">例子</a>|
| -b     | -&zwnj;-boot       | string    | The user:pass for bootstrap admin which is required to build default config file  | <a href="#start_ca_server">例子</a> |
|  | -&zwnj;-ca.certfile | string    | PEM-encoded CA certificate file (default "ca-cert.pem")  | <a name="锚点名称">例子</a> |
| -H | -&zwnj;-home | string    | 服务端home目录，默认当前目录 | <a name="锚点名称">例子</a> | 



<a name="init_ca_server">用以下的命令初始化fabirc-ca-server，初始化命令不会启动ca-server，只会生成相应的文件 </a>

```init
fabric-ca-server init -b admin:adminpw -H /root/ca-server/
```


<a name="start_ca_server">用以下的命令启动fabirc-ca-server，如果该服务器没有启动过，则等同于先初始化，再启动 </a>

```start
fabric-ca-server start -b admin:adminpw -H /root/ca-server/
```

初始化命令会在 /root/ca-server目录下生成以下文件：

```files
-rw-r--r--. 1 root root 19797 Jul 18 14:14 fabric-ca-server-config.yaml
drwxr-xr-x. 3 root root  4096 Jul 18 14:14 msp
-rw-r--r--. 1 root root   786 Jul 18 14:14 ca-cert.pem
-rw-r--r--. 1 root root   843 Jul 18 14:14 IssuerPublicKey
-rw-r--r--. 1 root root   215 Jul 18 14:14 IssuerRevocationPublicKey
-rw-r--r--. 1 root root 61440 Jul 18 14:14 fabric-ca-server.db
```



## Docker方式启动 fabric-ca-server

to do

# Hyberledger client

以下面的命令启动客户端

```client
export FABRIC_CA_CLIENT_HOME=/root/ca-client
fabric-ca-client enroll -u http://admin:adminpw@localhost:7054
```

## 参数说明

| 缩写       |  参数     | 参类型  | 说明      | 例子|
| ---------  | --------- | ----   | -------   | --- |
|      | -&zwnj;-address      | string    | fabric-ca-server 监听地址 (默认) "0.0.0.0")  | <a name="锚点名称">例子</a>|
| -b     | -&zwnj;-boot       | string    | The user:pass for bootstrap admin which is required to build default config file  | <a href="#start_ca_server">例子</a> |
|  | -&zwnj;-ca.certfile | string    | PEM-encoded CA certificate file (default "ca-cert.pem")  | <a name="锚点名称">例子</a> |
| -H | -&zwnj;-home | string    | 服务端home目录，默认当前目录 | <a name="锚点名称">例子</a> | 




- 注册新用户

下面的命令是用admin账户来注册一个新的账户 "admin2"，所属的机构是"org1.department1", ":ecert"后缀表明这个admin属性将会插入这个账户的enrollment certificat. 然后用与权限控制。 suffix means that by default the “admin” attribute and its value will be inserted into the identity’s enrollment certificate, which can then be used to make access control decisions.

maxenrollments 参数需要进行验证



```register
export FABRIC_CA_CLIENT_HOME=/root/ca-client
fabric-ca-client register --id.name admin2 --id.affiliation org1.department1 --id.attrs 'hf.Revoker=true,admin=true:ecert'
```

执行上面的命令之后，会看到有密码的输出，密码是自动生成的，类是这样的输出：

```passwdprint
Password: ziGRSLUqBXPT
```



- 用户登录



### 所属三级目录

## 另外一个二级目录