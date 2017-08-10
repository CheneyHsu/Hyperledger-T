# Vagrant Setup Fabric 1.0 Dev

# 核心组件
    Virtual Box
    Vagrant
    Git

# 环境准备
    1. 安装Virtual Box
    2. Vagrant
    3. Git
# 部署前准备
#### Git clone fabric，fabric-ca，fabric-sample，blockchain-monitoring
    git clone https://github.com/hyperledger/fabric
    git clone https://github.com/hyperledger/fabric-ca
    git clone https://github.com/hyperledger/fabric-samples
    git clone https://github.com/hyperledger/blockchain-monitoring

## Fabric 1.0 修改相关配置文件
#### 1. 替换gotools编译和clean所需要的tools（google连不上）

        进入fabric/devenv，在83行下增加
    
        #down x/tools first
        cat << EOF
        down tools first
        EOF

        tools=$GOPATH'/src/github.com/golang/tools'
        echo "tools path "$tools
        if [ ! -d "$tools" ]
        then
            sudo mkdir -p $tools
            sudo chown -R ubuntu:ubuntu $tools
            cd $GOPATH/src/github.com/golang
            git clone https://github.com/golang/tools.git
        else
            echo "already exist"
        fi

#### 2. 修改 fabric/gotools 目录下的Makefile，在44行下方增加
   
    @mkdir -p $(TMP_GOPATH)/src/github.com/golang/protobuf/
	@cp -R $(GOPATH)/src/github.com/hyperledger/fabric/vendor/github.com/golang/protobuf/* $(TMP_GOPATH)/src/github.com/golang/protobuf
#### 3. 在 $(GOBIN)/%: 22行下增加

    @echo "copy x/tools resource"
    @sudo mkdir -p $(TMP_GOPATH)/src/golang.org/x
    @sudo chown ubuntu.ubuntu -R $(TMP_GOPATH)/src/golang.org
    @cp -rf $(GOPATH)/src/github.com/golang/tools $(TMP_GOPATH)/src/golang.org/x
#### 4. 修改Vagrantfile文件添加要传输的文件夹
    if File.exist?("../../fabric-ca")
        config.vm.synced_folder "../../fabric-ca", "/opt/gopath/src/github.com/hyperledger/fabric-ca"
     end
    if File.exist?("../../fabric-samples")
        config.vm.synced_folder "../../fabric-samples", "/opt/gopath/src/github.com/hyperledger/fabric-samples"
     end
    if File.exist?("../../blockchain-monitoring")
        config.vm.synced_folder "../../blockchain-monitoring", "/opt/gopath/src/github.com/hyperledger/blockchain-monitoring"
     end

## Fabric-samples 修改相关配置文件（添加blockchain-monitoring）

#### 1. fabric-samples 取消GRPCS通信
修改 balance-transfer/config.json
    
    gprcs 更改为 grpc
修改 balance-transfer/app/network-config.json

    grpcs 更改为 grpc

修改 balance-transfer/artifacts/docker-compose.yml,增加如下内容：

        monitoring:
        container_name: blockchain-monitoring
            image: blockchainmonitoring/blockchain-monitoring:latest
            volumes:
            - ./channel/crypto-config/peerOrganizations:/opt/blockchain-monitoring/certs/peerOrganizations/:rw
            - ./net-config.yaml:/etc/conf/net-config.yaml
        # if you want customize configuration grafana or influxdb
        # - ./influxdb.conf:/etc/influxdb/influxdb.conf
        # - ./config/grafana/grafana.ini:/etc/grafana/grafana.ini
            environment:
        # SCHEDULED_TASKS_DELAY - defaule is 1000 milliseconds
            SCHEDULED_TASKS_DELAY: 10000
        # TIME_EVENT_LIFETIME - defaule is 1 hour
            TIME_EVENT_LIFETIME: "00:01:00"
            ports:
            - "3000:3000"
            - "8086:8086"
            - "5006:5005" #debug port
            depends_on:
            - ca.org1.example.com
            - ca.org2.example.com
            - peer0.org1.example.com
            - peer1.org1.example.com
            - peer0.org2.example.com
            - peer1.org2.example.com
            - orderer.example.com
修改balance-transfer/artifacts/base.yaml,如下：

        version: '2'
        services:
        peer-base:
            image: hyperledger/fabric-peer:x86_64-1.0.0
            environment:
            - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
            # the following setting starts chaincode containers on the same
            # bridge network as the peers
            # https://docs.docker.com/compose/networking/
            - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=artifacts_default
            - CORE_LOGGING_LEVEL=DEBUG
            - CORE_PEER_GOSSIP_USELEADERELECTION=true
            - CORE_PEER_GOSSIP_ORGLEADER=false
            # The following setting skips the gossip handshake since we are
            # are not doing mutual TLS
            - CORE_PEER_GOSSIP_SKIPHANDSHAKE=true
            - CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/crypto/peer/msp
            - CORE_PEER_TLS_ENABLED=false
            #- CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/crypto/peer/tls/server.key
            #- CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/crypto/peer/tls/server.crt
            #- CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/crypto/peer/tls/ca.crt
            working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
            command: peer node start
            volumes:
                - /var/run/:/host/var/run/

添加balance-transfer/artifacts/net-config.yaml配置文件如下：

        organisations:
        - name: 'Org1'
        msp: 'Org1MSP'
        member:
            login: 'admin'
            password: 'adminpw'
        ca:
            name: 'ca.org1.example.com'
            address: 'http://ca.org1.example.com:7054'
        peers:
            - name: 'peer0.org1.example.com'
            address: 'grpc://peer0.org1.example.com:7051'
            admin:
                login: 'admin'
                privkey: /opt/blockchain-monitoring/certs/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/5890f0061619c06fb29dea8cb304edecc020fe63f41a6db109f1e227cc1cb2a8_sk
                cert:   /opt/blockchain-monitoring/certs/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem
            - name: 'peer1.org1.example.com'
            address: 'grpc://peer1.org1.example.com:7051'
            admin:
                login: 'admin'
                privkey: /opt/blockchain-monitoring/certs/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/5890f0061619c06fb29dea8cb304edecc020fe63f41a6db109f1e227cc1cb2a8_sk
                cert:    /opt/blockchain-monitoring/certs/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem

        - name:  'Org2'
        msp: 'Org2MSP'
        member:
            login:  'admin'
            password:  'admin'
        ca:
            name: 'ca.org2.example.com'
            address: 'http://ca.org2.example.com:7054'
        peers:
            - name: 'peer0.org2.example.com'
            address: 'grpc://peer0.org2.example.com:7051'
            admin:
                login: 'admin'
                privkey: /opt/blockchain-monitoring/certs/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/keystore/1995b11d6573ed3be52fcd7a5fa477bc0f183e1f5f398c8281d0ce7c2c75a076_sk
                cert:    /opt/blockchain-monitoring/certs/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/Admin@org2.example.com-cert.pem
        peers:
            - name: 'peer1.org2.example.com'
            address: 'grpc://peer1.org2.example.com:7051'
            admin:
                login: 'admin'
                privkey: /opt/blockchain-monitoring/certs/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/keystore/1995b11d6573ed3be52fcd7a5fa477bc0f183e1f5f398c8281d0ce7c2c75a076_sk
                cert:    /opt/blockchain-monitoring/certs/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/Admin@org2.example.com-cert.pem

        channels:
        - name: 'mychannel'
        msp:
        - 'Org1MSP'
        - 'Org2MSP'

        endorsers:
        - name: 'peer0.org1.example.com'
            msp: 'Org1MSP'
            address: 'grpc://peer0.org1.example.com:7051'

        - name: 'peer1.org1.example.com'
            msp: 'Org1MSP'
            address: 'grpc://peer1.org1.example.com:7051'

        - name: 'peer0.org2.example.com'
            msp: 'Org2MSP'
            address: 'grpc://peer0.org2.example.com:7051'

        - name: 'peer1.org2.example.com'
            msp: 'Org2MSP'
            address: 'grpc://peer1.org2.example.com:7051'

        orderers:
        - name: 'orderer.example.com'
            msp:
            - 'Org1MSP'
            - 'Org2MSP'
            address: 'grpc://orderer.example.com:7050'

        events:
        - name: 'peer0.org1.example.com'
            msp: 'Org1MSP'
            address: 'grpc://peer0.org1.example.com:7053'

        - name: 'peer1.org1.example.com'
            msp: 'Org1MSP'
            address: 'grpc://peer1.org1.example.com:7053'

        - name: 'peer0.org2.example.com'
            msp: 'Org2MSP'
            address: 'grpc://peer0.org2.example.com:7053'

        - name: 'peer1.org2.example.com'
            msp: 'Org2MSP'
            address: 'grpc://peer1.org2.example.com:7053'

        chaincodes:
        - name: 'mycc'
            path: 'github.com/example_cc'
            version: 'v0'

## 使用vagrant进行开发环境部署
    cd fabric/devev   
    vagrant up
             

