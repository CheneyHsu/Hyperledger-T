# Setup Fabric 1.0 ENV

### OS Version

    CentOS 7.3 + Fabric 1.0 release

### Core components

    1.    Docker-compose:Docker 容器管理；
    2.    Go lang SDK:Go 语言开发、编译环境；
    3.    Git:git 镜像克隆与提交；
    4.    Rest Client: rest API 测试；

### System environment preparation
    1. sudo apt-get install curl vim
    2. sudo apt-get install git
    3. sudo apt-get install golang
    4. curl -sSL https://get.daocloud.io/docker | sh
    5. sudo usermod -aG docker hsukk;newgrp - docker
    6. sudo vi /etc/default/docker
    7. DOCKER_OPTS="$DOCKER_OPTS -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --api-cors-header='*'"
    8. curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://5c609c19.m.daocloud.io
    9. sudo service docker restart
    10. sudo systemctl enable docker.service
    11. sudo apt-get install python-pip
    12. curl -L https://get.daocloud.io/docker/compose/releases/download/1.10.1/docker-compose-`uname -s`-`uname -m` > ~/docker-compose
    13. sudo mv ~/docker-compose /usr/local/bin/docker-compose
    14. chmod +x /usr/local/bin/docker-compose
### Setup GoLang

    1.    wget https://storage.googleapis.com/golang/go1.8.1.linux-amd64.tar.gz
    2.    tar -xzvf go1.8.1.linux-amd64.tar.gz
    3.    mv ./go  /usr/local
    4.    vim /etc/profile
            export GOROOT=/usr/local/go
            export PATH=$PATH:$GOROOT/bin
    5.    source /etc/profile

### Setup Fabric 1.0 environment preparation

        1.    vim ~/.bashrc
                    export GOPATH=/home/hsukk/fabric
        2.    source ~/.bashrc
        4.    mkdir -p /home/hsukk/fabric/src/github.com/hyperledger
        5.    cd $GOPATH/src/github.com/hyperledger
        6.    git clone https://github.com/hyperledger/fabric.git
### Make gotools

        1.    mkdir mkdir -p  $GOPATH/src/github.com/hyperledger/fabric/gotools/build/gopath/src/golang.org/x/
        2. cd $GOPATH/src/github.com/hyperledger/fabric/gotools/build/gopath/src/golang.org/x/
        3.    git clone https://github.com/golang/tools
        4.    cd $GOPATH/src/github.com/hyperledger/fabric
        5.    make gotools
### Make Docker（Base）

        1.    cd $GOPATH/src/github.com/hyperledger/fabric
        2.    make docker
### >>>Not found proto-gen-go

        1.    cd $GOPATH/src/github.com/hyplerledger/fabric
        2.    cp -rf gotools/build/gopath/bin/protoc-gen-go ./build/docker/gotools/bin/
### Make docker (Fabric CA)

        1.    cd $GOPATH/src/github.com/hyperledger/fabric/
        2.    git clone https://github.com/hyperledger/fabric-ca.git
        3.    cd fabric-ca
        4.    make docker
