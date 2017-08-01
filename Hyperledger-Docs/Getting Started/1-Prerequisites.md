## Install cURL

安装最新的curl：CentOS-> yum  ; Ubuntu -> apt-get


#### Docker and Docker Compose

必须安装对应平台的的docker和docker-compose 来开发 Hyperledger Fabric:

* MacOSX, *nix, or Windows 10: Docker版本最低 17.03.0-ce 或更高.

可以使用如下命令在终端进行检查:

    docker --version

可以使用如下命令在终端进行检查，docker-compose版本1.8或更高:

    docker-compose --version

## Go Programming Language

Hyperledger Fabric使用Go语言1.7版本和许多组件。

编写一个Go chaincode 程序，需要$GOPATH支持，务必确保源码位于$GOPATH内，首先要检查你的$GOPATH

    echo $GOPATH
    /Users/xxx/go

如果没有设置GOPATH的环境变量，需要在 `~/.bashrc`中进行添加:

    export GOPATH=$HOME/go
    export PATH=$PATH:$GOPATH/bin

## Node.js Runtime and NPM

如果开发程序时使用Node.js，并使用Hyperledger Fabric SDK for Node.js, Node.js版本必须是 6.9.x of Node.js installed.

>Node.js version 7.x 不支持

`Node.js - version 6.9.x or greater`

安装NodeJS需要安装NPM，先使用一下命令进行npm版本确认和升级

    npm install npm@3.10.10 -g
