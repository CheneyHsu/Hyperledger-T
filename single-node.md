# Setup  Fabric 1.0  （SingleNode E2E）

### OS Version

```
CentOS 7.3 + Fabric 1.0 release
```

### Core components

```
1.    Docker-compose:Docker 容器管理；   
2.    Go lang SDK:Go 语言开发、编译环境；     
3.    Git:git 镜像克隆与提交；     
4.    Rest Client: rest API 测试；
```

### System environment preparation

```
1.    yum update
2.    yum install –y  docker gcc gcc-c++ epel-release
3.    yum install docker
4.    systemctl enable docker
5.    systemctl restart docker
6.    yum -y install python-pip
7.    pip install docker-compose>=1.8.0
8.    yum -y install git
9.    yum install -y libtool libltdl-dev openssl
10.   setenforce 0
```

```
1. Setup Selinux to permissive or disabled 
    vim /etc/selinux/config
    SELINUX=enforcing -->  SELINUX=disabled
```

### Setup GoLang

```
1.    wget https://storage.googleapis.com/golang/go1.8.1.linux-amd64.tar.gz
2.    tar -xzvf go1.8.1.linux-amd64.tar.gz
3.    mv ./go  /usr/local
4.    vim /etc/profile
            export GOROOT=/usr/local/go
            export PATH=$PATH:$GOROOT/bin
5.    source /etc/profile
```

### Setup Fabric 1.0 environment preparation

```
1.    vim ~/.bashrc
            export GOPATH=/home/fabric/fabric
2.    source ~/.bashrc
3.    cd $GOPATH
4.    mkdir -p src/github.com/hyperledger 
5.    cd $GOPATH/src/github.com/hyperledger
6.    git clone https://github.com/hyperledger/fabric.git
```

### Make gotools

```
1.    cd $GOPATH/src/github.com/hyperledger/fabric/gotools/build/gopath/src/github.com/golang.org/x/
        #if not directory , maybe use mkdir create it. 
2.    git clone https://github.com/golang/tools
3.    cd /root/fabric/fabric/src/github.com/hyperledger/fabric
4.    make gotools
```

### Make Docker（Base）

```
1.    cd $GOPATH/src/github.com/hyperledger/fabric
2.    make docker
```

### Not found proto-gen-go

```
1.    cd $GOPATH
2.    cp -rf gotools/build/gopath/bin/protoc-gen-go ./build/docker/gotools/bin/
```

### Make docker \(Fabric CA\)

```
1.    cd $GOPATH/src/github.com/hyperledger/fabric/
2.    git clone https://github.com/hyperledger/fabric-ca.git
3.    cd fabric-ca
4.    make docker
```

### Startup E2E

```

```



