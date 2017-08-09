# Building Your First Network

>仅在sample中使用命令，如果使用当前主分支的图像或工具运行这些命令，您可能会看到配置和panic错误

创建第一个网络方案，维护2个`组织`，以solo模式运行，保持2个peer.

### Install prerequisites

开始之前，检查你的所有必须条件，在Hyperledger fabric后续可能用到的开发和操作必须条件。

下载并安装 Hyperledger Fabric Samples. 你会发现许多内容. 打开子目录，使用第一个网络示例.

      cd first-network

该文档中提供的命令必须从克隆目录的第一个网络子目录运行，如果选择从另外一个位置运行命令，则命令将会报错。

### 运行试试？？

我们提供了一个完全注释的脚本byfn.sh。利用这个脚本将很快组成4个节点的网络，并且是不同组织的链接。
同时也将利用一个容器来执行脚本，完成渠道部署，和实例化链码和进行交易。

 byfn.sh 帮助如下:

./byfn.sh -h

    Usage:
    byfn.sh -m up|down|restart|generate [-c <channel name>] [-t <timeout>]
    byfn.sh -h|--help (print this message)
    -m <mode> - one of 'up', 'down', 'restart' or 'generate'
    - 'up' - bring up the network with docker-compose up
    - 'down' - clear the network with docker-compose down
    - 'restart' - restart the network
    - 'generate' - generate required certificates and genesis block
    -c <channel name> - config name to use (defaults to "mychannel")
    -t <timeout> - CLI timeout duration in microseconds (defaults to 10000)

通常，首先生成所需的证书和生成区块，而后提出网络，例如：

    byfn.sh -m generate -c <channelname>
    byfn.sh -m up -c <channelname>

If you choose not to supply a channel name, then the script will use a default name of mychannel. The CLI timeout parameter (specified with the -t flag) is an optional value; if you choose not to set it, then your CLI container will exit upon conclusion of the script.
如果你选择不提供通道名称，mychannel将作为默认名称，Cli的超时设置是一个可选值，如果不设置它，那么脚本结束时，cli容器将退出。

### Generate Network Artifacts

主备好了吗？开始!
命令如下：

    ./byfn.sh -m generate

接受默认设定值，输入Y进行确认!

    Generating certs and genesis block for with channel 'mychannel' and CLI timeout of '10000'
    Continue (y/n)?y
    proceeding ...
    /Users/xxx/dev/fabric-samples/bin/cryptogen

    ##########################################################
    ##### Generate certificates using cryptogen tool #########
    ##########################################################
    org1.example.com
    2017-06-12 21:01:37.334 EDT [bccsp] GetDefault -> WARN 001 Before using BCCSP, please call InitFactories(). Falling back to bootBCCSP.
    ...

    /Users/xxx/dev/fabric-samples/bin/configtxgen
    ##########################################################
    #########  Generating Orderer Genesis block ##############
    ##########################################################
    2017-06-12 21:01:37.558 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
    2017-06-12 21:01:37.562 EDT [msp] getMspConfig -> INFO 002 intermediate certs folder not found at [/Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts]. Skipping.: [stat /Users/xxx/dev/byfn/crypto-config/ordererOrganizations/example.com/msp/intermediatecerts: no such file or directory]
    ...
    2017-06-12 21:01:37.588 EDT [common/configtx/tool] doOutputBlock -> INFO 00b Generating genesis block
    2017-06-12 21:01:37.590 EDT [common/configtx/tool] doOutputBlock -> INFO 00c Writing genesis block

    #################################################################
    ### Generating channel configuration transaction 'channel.tx' ###
    #################################################################
    2017-06-12 21:01:37.634 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
    2017-06-12 21:01:37.644 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 002 Generating new channel configtx
    2017-06-12 21:01:37.645 EDT [common/configtx/tool] doOutputChannelCreateTx -> INFO 003 Writing new channel tx

    #################################################################
    #######    Generating anchor peer update for Org1MSP   ##########
    #################################################################
    2017-06-12 21:01:37.674 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
    2017-06-12 21:01:37.678 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
    2017-06-12 21:01:37.679 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update

    #################################################################
    #######    Generating anchor peer update for Org2MSP   ##########
    #################################################################
    2017-06-12 21:01:37.700 EDT [common/configtx/tool] main -> INFO 001 Loading configuration
    2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 002 Generating anchor peer update
    2017-06-12 21:01:37.704 EDT [common/configtx/tool] doOutputAnchorPeersUpdate -> INFO 003 Writing anchor peer update



第一步为我们的所有网络实体生成所有证书和密钥，用于引导订单服务的区块生成，以及配置通道所需的配置事务集合。

### Bring Up the Network

使用以下命令启动网络

    ./byfn.sh -m up

再一次回答 y进行确认默认的设定值.

    Starting with channel 'mychannel' and CLI timeout of '10000'
    Continue (y/n)?y
    proceeding ...
    Creating network "net_byfn" with the default driver
    Creating peer0.org1.example.com
    Creating peer1.org1.example.com
    Creating peer0.org2.example.com
    Creating orderer.example.com
    Creating peer1.org2.example.com
    Creating cli


     ____    _____      _      ____    _____
    / ___|  |_   _|    / \    |  _ \  |_   _|
    \___ \    | |     / _ \   | |_) |   | |
     ___) |   | |    / ___ \  |  _ <    | |
    |____/    |_|   /_/   \_\ |_| \_\   |_|

    Channel name : mychannel
    Creating channel...

日志还在继续.这些容器会被启动，完成一个端到端的场景（E2E），完成后，将会有如下报告：

    2017-05-16 17:08:01.366 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
    2017-05-16 17:08:01.366 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
    2017-05-16 17:08:01.366 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AB1070A6708031A0C08F1E3ECC80510...6D7963631A0A0A0571756572790A0161
    2017-05-16 17:08:01.367 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: E61DB37F4E8B0D32C9FE10E3936BA9B8CD278FAA1F3320B08712164248285C54
    Query Result: 90
    2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
    ===================== Query on PEER3 on channel 'mychannel' is successful =====================

    ===================== All GOOD, BYFN execution completed =====================


     _____   _   _   ____
    | ____| | \ | | |  _ \
    |  _|   |  \| | | | | |
    | |___  | |\  | | |_| |
    |_____| |_| \_| |____/

可以使用滚动来查看日志和各种事物，如果没有这个结果，那么到故障分析部分，让我们看看能否发现什么问题。

### Bring Down the Network

最后清楚所有容器和文件，还原到启动之前的状态。

    ./byfn.sh -m down

在一次使用y确认默认的设定值:

    Stopping with channel 'mychannel' and CLI timeout of '10000'
    Continue (y/n)?y
    proceeding ...
    WARNING: The CHANNEL_NAME variable is not set. Defaulting to a blank string.
    WARNING: The TIMEOUT variable is not set. Defaulting to a blank string.
    Removing network net_byfn
    468aaa6201ed
    ...
    Untagged: dev-peer1.org2.example.com-mycc-1.0:latest
    Deleted: sha256:ed3230614e64e1c83e510c0c282e982d2b06d148b1c498bbdcc429e2b2531e91
    ...

如果你想学习更多东西，还请继续往下阅读，将会分解这些步骤的组成和使用.
### Crypto Generator

使用cryptogen进行证书生成（X509证书），这些证书是身份的代表，将允许在我们的实体通讯和处理时进行签名和身份验证。

### How does it work?

cryptogen 元素来源于 crypto-config.yaml，
该文件包含网络拓扑，允许我们为范围内包含的组织和组件生成证书和密钥。每个组织都提供一个独立的根证书（CA证书），结合这个组织特定的组件 (peers and orderers)，通过给每个组织分配一个唯一的CA证书，我们模拟了一个典型网络，其中一个参与成员将使用它自己的授权证书在Hyperledger中进行通讯交易，并且通信是由一个实体的私钥签名，然后通过公钥进行验证。

该文件中有一个计数变量，我们将使用这个数值来指定每个组织中的peers，在我们的例子中，每个组织有2个peers，我们不去深入讨论x509和公钥的基础知识，如有你有兴趣，可以去参考其他主题。

运行工具之前，让我们分析一下crypto-config.yaml。特别注意“Name”, “Domain” and “Specs” :

    OrdererOrgs:
    #---------------------------------------------------------
    # Orderer
    # --------------------------------------------------------
    - Name: Orderer
      Domain: example.com
      # ------------------------------------------------------
      # "Specs" - See PeerOrgs below for complete description
    # -----------------------------------------------------
      Specs:
        - Hostname: orderer
    # -------------------------------------------------------
    # "PeerOrgs" - Definition of organizations managing peer nodes
    # ------------------------------------------------------
    PeerOrgs:
    # -----------------------------------------------------
    # Org1
    # ----------------------------------------------------
    - Name: Org1
      Domain: org1.example.com

网络命名格式约定为 “{{.Hostname}}.{{.Domain}}”. 所以按约定命名约定唯一的MSP ID为orderer.example.com. 这个文档包含大量的定义和语法，可以参考程序提供（MSP）文档来深入了解

在我们运行cryptogen工具生成的证书和密钥将被保存到一个名为crypto-config的文件夹中.

### Configuration Transaction Generator

 `configtxgen` 工具用于创建4个组件:

        * orderer genesis block,
        * channel channel configuration transaction,
        * and two anchor peer transactions - one for each Peer Org.

参考channel 配置文件 configtxgen

块的排序是在创世块之后由排序服务支持，在通道创建时的交易事物文件会广播到orderer。正如名称建议，peer节点的交易将在这个通道上指定每个Org上的peer节点。

### How does it work?

Configtxgen 元素来源于`configtx.yaml`文件，它包含示例网络的定义， 有三个成员，一个是orderer（Org （OrdererOrg））和2个peer节点，分布在Org1和Org2中。
该文件还指定了一个共同体`SampleConsortium` ，有2个对等的组织组成.特别注意文件的“Profiles”部分， 你会注意到有2处不同，一是orderer的创世块`TwoOrgsOrdererGenesis`,另外一个是channel部分`TwoOrgsChannel`

这些都和你重要，因为创建时这些都将作为参数被传递。

>Notice that our SampleConsortium is defined in the system-level profile and then referenced by our channel-level profile. Channels exist within the purview of a consortium, and all consortia must be defined in the scope of the network at large.

这个文件包含2个比较值得关注的规范，首先需要为每个peers指定org(peer0.org1.example.com & peer0.org2.example.com)。
其次我们为每个成员指定MSP的目录，相应的允许存储的根证书在每个组织的创世块中，这是个关键的概念，现在，任何与ordering service通信的网络实体都可以对其数字签名进行验证。

### Run the tools

你可以手动生成证书和秘钥和配置，使用onfigtxgen and cryptogen 命令，或者你可以尝试修改`byfn.sh `.

你可以为要生成的证书配置到`crypto-config.yaml`文件中，定义到`byfn.sh`脚本`generateCerts`功能.仅作参考


首先来运行`cryptogen tool`. 执行文件在`bin`目录, 所以需要提供文件的相对路径.

    ../bin/cryptogen generate --config=./crypto-config.yaml


你将看到如下警告输出,这些警告没有影响,忽略掉..

    [bccsp] GetDefault -> WARN 001 Before using BCCSP, please call InitFactories(). Falling back to bootBCCSP.

接下来需要告诉`configtxgen`工具哪里找到`onfigtx.yaml`,


 为`configtx.yaml`设置环境变量

export FABRIC_CFG_PATH=$PWD

使用`configtxgen` 创建创始块:

    ../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

sample network 中没有使用这些示例,你可以忽略掉这些警告日志.

接下来,我们来创建channel,需要设置channel_name的环境变量

    export CHANNEL_NAME=mychannel

>this file contains the definitions for our sample channel
    ../bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

接下来,我们将定义Org1,注意环境变量Channel_name.

    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP

现在定义Org2:

    ../bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP

### Start the network

利用docker-compose 脚本启动我们的网络.docker-compose文件将会下载docker images并启动orderer以及创建创世块.

    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
    volumes

如果没有进行注释,脚本将执行所有命令,并且为我们描述后台日志记录,但是我们希望手动执行命令,以了解每个调用语法和功能.

超时变量,Cli容器退出时间

Start your network:

    CHANNEL_NAME=$CHANNEL_NAME TIMEOUT=<pick_a_value> docker-compose -f docker-compose-cli.yaml up -d

如果你想实时查看日志,则不要`-d`参数,让日志打印在终端,但是需要打开第二个终端来执行cli.

### Environment variables

下面使用CLI对peer0.org1.example.com进行操作, 我们需要用下面给出四个环境变量来描述我们的命令 . 环境变量在peer0.org1.example.com 容器中,然而, 如果你想发送指令去其他的peers或者 orderer, 必须事先准备环境变量. 检查 docker-compose-base.yaml 文件:

    # Environment variables for PEER0

    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    CORE_PEER_ADDRESS=peer0.org1.example.com:7051
    CORE_PEER_LOCALMSPID="Org1MSP"
    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

### Create & Join Channel

使用如下指令进入CLI容器

    docker exec -it cli bash

如果成功将显示如下:

    root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#


记得前面利用configtxgen加工过 channel.tx 文件，这个文件将是我们创建channel的必要一部分。

--cafile 参数做为命令的一部分，必须要保障使用根证书，完成TLS握手。

使用-c参数指定channel名称，使用-f指定channel.tx

    export CHANNEL_NAME=mychannel

    # the channel.tx file is mounted in the channel-artifacts directory within your CLI container
    # as a result, we pass the full path for the file
    # we also pass the path for the orderer ca-cert in order to verify the TLS handshake
    # be sure to replace the $CHANNEL_NAME variable appropriately

    peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem

这个命令将创建创世块<channel-ID.block>
加入这个频道，配置信息包含在channel.tx.

将 peer0.org1.example.com 加入channel.

    # By default, this joins ``peer0.org1.example.com`` only
    # the <channel-ID.block> was returned by the previous command

     peer channel join -b <channel-ID.block>


设置4个环境变量可以使其他的perr加入channel.

### Install & Instantiate Chaincode

我们将使用一个简单的链码进行测试，如何编写链码，参考后面的教程.

应用程序的交互通过区块链分类的链码,因此我们需要对在每个节点去部署链码,然后初始化链码,来支持交易.

第一步,安装简单的go code在你的4个peer nodes上,这些源码在peer的文件系统上.

    peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02

接下来,初始化channel 上的链码,通过链码集初始化链码,并且设置初始化参数,注意`-p`参数, 交易的背书策略对于一个交易的验证.

下面的命令中,指定背书策略为

    <<<<<<< HEAD
    -P "OR ('Org0MSP.member','Org1MSP.member')".
    意义是需要背书在Org1 和 Org2 的每个peer。可以改变下语法试着需要2个背书`AND`


>详见背书策略！
## Query

我们查询一个正确实例化的链码，通过state DB，语法如下：

        # be sure to set the -C and -n flags appropriately

        peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

## Invoke

现在，我们从将10从A移动到B，这个事务将生成一个新的块并更新状态数据库，语法如下：

        # be sure to set the -C and -n flags appropriately

        peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'

## Query

确认并查询记录，原有A是100.

        # be sure to set the -C and -n flags appropriately

        peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

We should see the following:

        Query Result: 90


## What’s happening behind the scenes?
`./byfn.sh -m down`清除所有环境和记录

* 脚本script.sh安装在`CLI`容器内，通过`CreateChanel`命令和频道名称以及`channel.tx`文件来做channel配置。

* CreateChannel会创建创世块`genesis block` - `<your_channel_name>.block` -存储在peer容器的文件系统上，通过容器channel配置文件`channel.tx`指定
 
* 所有的 peers都需要执行join，指定 <your_channel_name> 和 <your_channel_name>.block.
* 现在由4个perr组成channel和2个组织，2个组织的配置文件如下<TwoOrgsChannel>.
        
        peer0.org1.example.com and peer1.org1.example.com belong to Org1; peer0.org2.example.com and peer1.org2.example.com belong to Org2
* 这些关系都在`crypto-config.yaml`定义以及`MSP`路径在docker compose中指定.

* Org1MSP的Anchor
Peer(peer0.org1.example.com)和Org2MSP的Anchor
peer（peer0.org2.example.com）被更新。我们通过传递Org1MSPanchor.tx和Org2MSPanchor.tx以及channel的名字到ordering
service来实现这一步。

* 将编写好的chaincode_example02安装在peer0.org1.example.com和peer0.org2.example.com上（这里并没有安装在所有peer上，而是仅安装在anchor
peer节点上，anchor
peer节点之前每个Org设置了一个）
* chaincode在peer0.org2.example.com上实例化。实例化过程添加chaincode到channel中，为目标peer启动容器，同时初始化与chaincode相关的key-value键值对。本例中初始化的值为[“a”,”100”
“b”,”200”]。实例化后会启动一个容器dev-peer0.org2.example.com-mycc-1.0。
* 实例化过程也传递了一个背书策略的参数。背书策略类似形式：-P "OR ('Org1MSP.member','Org2MSP.member')"，代表任何交易必须被Org1或Org2的一个peer背书。
* 在peer0.org1.example.com上执行查询a的值。chaincode之前已经安装在peer0.org1.example.com上了，因此查询操作会为Org1的peer0节点启动一个容器dev-peer0.org1.example.com-mycc-1.0。查询结果也会返回回来，这个过程中没有任何写操作发生，所以a的值还是100。
    
* 发送一个转移账户金额的调用到peer0.org1.example.com，从a账户转移10单位至b账户
    
* chaincode然后安装在peer1.org2.example.com上

*发送查询a账户操作至peer1.org2.example.com。这将启动第三个chaincode容器dev-peer1.org2.example.com-mycc-1.0。返回金额90，说明之前帐号金额的转移操作成功执行。.

## 这说明了什么

为了在账本上成功的执行读写操作，chaincode必须安装在peer上。另外，chaincode容器直到实例化或者传统交易-读写执行的时候（例：查询a账户的值），chaincode容器才会启动。channel中的每个节点都维护了账本的完全复制，存储了不可改变的、序列化的记录区块以及state
database用于保存当前的fabric状态。即便是那些没有安装chaincode的节点（例如peer1.org1.example.com）也会同步账本。最终chaincode在安装到peer1.org1.example.com后就可以被调用了，因为chaincode已经完成了实例化。
## 怎样查看交易信息

C查看CLI
docker容器的日志信息
    docker logs -f cli

可以看到交易的详细过程

    2017-05-16 17:08:01.366 UTC [msp] GetLocalMSP -> DEBU 004 Returning existing local MSP
    2017-05-16 17:08:01.366 UTC [msp] GetDefaultSigningIdentity -> DEBU 005 Obtaining default signing identity
    2017-05-16 17:08:01.366 UTC [msp/identity] Sign -> DEBU 006 Sign: plaintext: 0AB1070A6708031A0C08F1E3ECC80510...6D7963631A0A0A0571756572790A0161
    2017-05-16 17:08:01.367 UTC [msp/identity] Sign -> DEBU 007 Sign: digest: E61DB37F4E8B0D32C9FE10E3936BA9B8CD278FAA1F3320B08712164248285C54
    Query Result: 90
    2017-05-16 17:08:15.158 UTC [main] main -> INFO 008 Exiting.....
    ===================== Query on PEER3 on channel 'mychannel' is successful =====================

    ===================== All GOOD, BYFN execution completed =====================


    _____   _   _   ____
    | ____| | \ | | |  _ \
    |  _|   |  \| | | | | |
    | |___  | |\  | | |_| |
    |_____| |_| \_| |____/

## 怎样查看chaincode的日志

在每个chaincode container上可以查看当前container里所执行过的交易。具体查看命令如下：

    $ docker logs dev-peer0.org2.example.com-mycc-1.0
    04:30:45.947 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
    ex02 Init
    Aval = 100, Bval = 200

    $ docker logs dev-peer0.org1.example.com-mycc-1.0
    04:31:10.569 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
    ex02 Invoke
    Query Response:{"Name":"a","Amount":"100"}
    ex02 Invoke
    Aval = 90, Bval = 210

    $ docker logs dev-peer1.org2.example.com-mycc-1.0
    04:31:30.420 [BCCSP_FACTORY] DEBU : Initialize BCCSP [SW]
    ex02 Invoke
    Query Response:{"Name":"a","Amount":"90"}

## 理解docker-compose拓扑结构

BYFN例子提供了两种docker-compose文件配置，每一种都是由docker-compose-base.yaml（文件存放在base文件夹中）文件拓展而来。第一个配置文件是docker-compose-cli.yaml，该配置文件配置了一个CLI容器，一个orderer，4个peer节点。使用该配置文件启动可以实现本文中的所有操作指令。第二种配置文件docker-compose-e2e.yaml是配置启动一个使用Node.js
SDK的点对点测试。这个配置文件的主要区别是包含了fabric-ca-servers容器。因此，我们可以使用REST接口实现向CA组织注册和登记用户。

如果你想使用docker-compose-e2e.yaml并且不先运行byfn.sh脚本，那么我们需要做4个微改动。我们需要设定Organization
CA的私鈅。你可以设定这些值为你的crypto-config文件夹。例如设置Org1的私鈅路径为：crypto-config/peerOrganizations/org1.example.com/ca/。私鈅文件是一个长hash值加上_sk组成。设定Org2的私鈅为crypto-config/peerOrganizations/org2.example.com/ca/

另外2出改动是修改docker-compose-e2e.yaml中ca0和ca1配置中的FABRIC_CA_SERVER_TLS_KEYFILE变量对应的值。需要指定tls证书所在的路径。

#### 按照一个简单区块链网络的生成过程，制作执行过程如下：
1. 生成必要信息

    1、网路节点的证书文件

    ../bin/cryptogen generate --config=./crypto-config.yaml

   2、生成gensis block

    export FABRIC_CFG_PATH=$PWD
    ../bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block

   3、生成channel.tx

    ../bin/configtxgen
			-profile TwoOrgsChannel -outputCreateChannelTx
			./channel-artifacts/channel.tx -channelID $CHANNEL_NAME

   4、生成anchor peer
    ../bin/configtxgen
			-profile TwoOrgsChannel -outputAnchorPeersUpdate
			./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME
			-asOrg Org1MSP

2. 创建channel

    peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx

    创建channel的命令由peer节点发起，使用上文生成的channel.tx，上述命令生成<channel_name>.block在当前文件夹中，该block中保存了channel.tx的信息。节点加入channel后，该区块作为节点区块链的第一个区块。

3. 加入peer节点到channel中

    peer channel join -b $CHANNEL_NAME.block

    peer加入channel时使用创建channel时生成的block文件

    加入哪个peer节点到channel中，需要通过在CLI容器中设置如下环境变量指定：

    CORE_PEER_LOCALMSPID="Org1MSP"

    CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt

    CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp


    CORE_PEER_ADDRESS=peer0.org1.example.com:7051

    设置上述环境变量，加入peer0节点到channel中

4. 更新Anchor peer
    
    peer channel update -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/${CORE_PEER_LOCALMSPID}anchors.tx

    使用configtxgen生成的anchor.tx文件

5. 安装chaincode
 	
    peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02

    在CLI容器中设置环境变量指定在哪个peer节点上安装chaincode

6. 在channel上实例化chaincode
    peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"

    需要设定环境变量指定在哪个peer节点上执行实例化chaincode命令

7. 执行chaincode调用


## Using CouchDB

状态数据库可以从默认的 (goleveldb) 切换到CouchDB. 相同的链码功能同样支持CouchDB, 然后, 可以提供更丰富和复杂的查询从状态数据库，建模为JSON.

使用 CouchDB 替代默认的 (goleveldb), 使用 docker-compose-couch.yaml配置文件来完成:

    CHANNEL_NAME=$CHANNEL_NAME TIMEOUT=<pick_a_value> docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d

#### chaincode_example02 现在可以工作在CouchDB.

你可以部署chaincode_example02链码在CouchDB状态数据库使用上面的步骤，但是为了联系CouchDB的查询功能，您将需要使用一个链码，建模JSON数据，（如marbles02）。你可以在本地找到marbles02链码，在 fabric/examples/chaincode/go directory.

我们将按照同样的过程创建和加入频道，如上文所述的创建和连接通道部分所述。一旦你加入peers的频道，使用下面的步骤与marbles02码互动： 

#### Install and instantiate the chaincode on peer0.org1.example.com:

    # be sure to modify the $CHANNEL_NAME variable accordingly for the instantiate command

    peer chaincode install -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles02
    peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.member','Org1MSP.member')"

#### Create some marbles and move them around:

# be sure to modify the $CHANNEL_NAME variable accordingly

    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble2","jerry"]}'
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'

使用浏览器访问如下URL:

    http://localhost:5984/_utils

你可以看见一个mychannel的数据库.

> 一下命令一定要更新 $CHANNEL_NAME 变量.

可以在 CLI 运行查询 (e.g. reading marble2):

    peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'

输出可以看到详细的信息marble2:

    Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}

检索历史 marble1:

    peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'

输出可以看到详细信息 marble1:

    Query Result: [{"TxId":"1c3d3caf124c89f91a4c0f353723ac736c58155325f02890adebaa15e16e6464", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}},{"TxId":"755d55c281889eaeebf405586f9e25d71d36eb3d35420af833a20a2f53a3eefd", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"jerry"}},{"TxId":"819451032d813dde6247f85e56a89262555e04f14788ee33e28b232eef36d98f", "Value":}]

查询jerry:

    peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'

查到2个jerry:

Query Result: [{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}},{"Key":"marble3", "Record":{"color":"blue","docType":"marble","name":"marble3","owner":"jerry","size":70}}]

#### 数据持久化

使CouchDB数据持久在 docker-compose-base.yaml 文件peer部分中添加如下:

    volumes:
    - /var/hyperledger/peer0:/var/hyperledger/production

在CouchDB部分添加如下:

    volumes:
    - /var/hyperledger/couchdb0:/opt/couchdb/data

## Troubleshooting

开始新的网络前，先使用如下命令清空项目，密码，容器以及链码。

    ./byfn.sh -m down

若果不能删除容器，可以看到报错。
如果是容器报错，首先检测你的版本（17.03.1以上），可以使用如下命令直接删除。

    docker rm -f $(docker ps -aq)
    docker rmi -f $(docker images -q)

如果错误是 create, instantiate, invoke or query 命令, 确保你有正确的更新频道名称和链码的名字。在提供的示例命令中有占位符值。 
    
如果你看到如下报错:

Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)

有可能是 (e.g. dev-peer1.org2.example.com-mycc-1.0 or dev-peer0.org1.example.com-mycc-1.0) 正在运行或者已经存在.删除后重试.

    docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')

Error connecting: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
Error: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure

版本需要时1.0.0，而且需要标记“latest”.


[configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
panic: Error reading configuration: Unsupported Config Type ""

环境变量设置错误 FABRIC_CFG_PATH，configtxgen工具变量来至configtx.yaml. 设置变量 FABRIC_CFG_PATH=$PWD, 然后重新创建channel.

清除所有设置

    ./byfn.sh -m down
