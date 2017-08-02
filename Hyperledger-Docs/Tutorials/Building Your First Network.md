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
  -P "OR ('Org0MSP.member','Org1MSP.member')".
  This means that we need “endorsement” from a peer belonging to Org1 OR Org2 (i.e. only one endorsement). If we changed the syntax to AND then we would need two endorsements.

# be sure to replace the $CHANNEL_NAME environment variable
# if you did not install your chaincode with a name of mycc, then modify that argument as well

peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"

See the endorsement policies documentation for more details on policy implementation.
Query

Let’s query for the value of a to make sure the chaincode was properly instantiated and the state DB was populated. The syntax for query is as follows:

# be sure to set the -C and -n flags appropriately

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

Invoke

Now let’s move 10 from a to b. This transaction will cut a new block and update the state DB. The syntax for invoke is as follows:

# be sure to set the -C and -n flags appropriately

peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'

Query

Let’s confirm that our previous invocation executed properly. We initialized the key a with a value of 100 and just removed 10 with our previous invocation. Therefore, a query against a should reveal 90. The syntax for query is as follows.

# be sure to set the -C and -n flags appropriately

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

We should see the following:

Query Result: 90

Feel free to start over and manipulate the key value pairs and subsequent invocations.
What’s happening behind the scenes?

Note

These steps describe the scenario in which script.sh is not commented out in the docker-compose-cli.yaml file. Clean your network with ./byfn.sh -m down and ensure this command is active. Then use the same docker-compose prompt to launch your network again

    A script - script.sh - is baked inside the CLI container. The script drives the createChannel command against the supplied channel name and uses the channel.tx file for channel configuration.
    The output of createChannel is a genesis block - <your_channel_name>.block - which gets stored on the peers’ file systems and contains the channel configuration specified from channel.tx.
    The joinChannel command is exercised for all four peers, which takes as input the previously generated genesis block. This command instructs the peers to join <your_channel_name> and create a chain starting with <your_channel_name>.block.
    Now we have a channel consisting of four peers, and two organizations. This is our TwoOrgsChannel profile.
    peer0.org1.example.com and peer1.org1.example.com belong to Org1; peer0.org2.example.com and peer1.org2.example.com belong to Org2
    These relationships are defined through the crypto-config.yaml and the MSP path is specified in our docker compose.
    The anchor peers for Org1MSP (peer0.org1.example.com) and Org2MSP (peer0.org2.example.com) are then updated. We do this by passing the Org1MSPanchors.tx and Org2MSPanchors.tx artifacts to the ordering service along with the name of our channel.
    A chaincode - chaincode_example02 - is installed on peer0.org1.example.com and peer0.org2.example.com
    The chaincode is then “instantiated” on peer0.org2.example.com. Instantiation adds the chaincode to the channel, starts the container for the target peer, and initializes the key value pairs associated with the chaincode. The initial values for this example are [“a”,”100” “b”,”200”]. This “instantiation” results in a container by the name of dev-peer0.org2.example.com-mycc-1.0 starting.
    The instantiation also passes in an argument for the endorsement policy. The policy is defined as -P "OR    ('Org1MSP.member','Org2MSP.member')", meaning that any transaction must be endorsed by a peer tied to Org1 or Org2.
    A query against the value of “a” is issued to peer0.org1.example.com. The chaincode was previously installed on peer0.org1.example.com, so this will start a container for Org1 peer0 by the name of dev-peer0.org1.example.com-mycc-1.0. The result of the query is also returned. No write operations have occurred, so a query against “a” will still return a value of “100”.
    An invoke is sent to peer0.org1.example.com to move “10” from “a” to “b”
    The chaincode is then installed on peer1.org2.example.com
    A query is sent to peer1.org2.example.com for the value of “a”. This starts a third chaincode container by the name of dev-peer1.org2.example.com-mycc-1.0. A value of 90 is returned, correctly reflecting the previous transaction during which the value for key “a” was modified by 10.

What does this demonstrate?

Chaincode MUST be installed on a peer in order for it to successfully perform read/write operations against the ledger. Furthermore, a chaincode container is not started for a peer until an init or traditional transaction - read/write - is performed against that chaincode (e.g. query for the value of “a”). The transaction causes the container to start. Also, all peers in a channel maintain an exact copy of the ledger which comprises the blockchain to store the immutable, sequenced record in blocks, as well as a state database to maintain a snapshot of the current state. This includes those peers that do not have chaincode installed on them (like peer1.org1.example.com in the above example) . Finally, the chaincode is accessible after it is installed (like peer1.org2.example.com in the above example) because it has already been instantiated.
How do I see these transactions?

Check the logs for the CLI Docker container.

docker logs -f cli

You should see the following output:

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

You can scroll through these logs to see the various transactions.
How can I see the chaincode logs?

Inspect the individual chaincode containers to see the separate transactions executed against each container. Here is the combined output from each container:

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

Understanding the Docker Compose topology

The BYFN sample offers us two flavors of Docker Compose files, both of which are extended from the docker-compose-base.yaml (located in the base folder). Our first flavor, docker-compose-cli.yaml, provides us with a CLI container, along with an orderer, four peers. We use this file for the entirety of the instructions on this page.

Note

the remainder of this section covers a docker-compose file designed for the SDK. Refer to the Node SDK repo for details on running these tests.

The second flavor, docker-compose-e2e.yaml, is constructed to run end-to-end tests using the Node.js SDK. Aside from functioning with the SDK, its primary differentiation is that there are containers for the fabric-ca servers. As a result, we are able to send REST calls to the organizational CAs for user registration and enrollment.

If you want to use the docker-compose-e2e.yaml without first running the byfn.sh script, then we will need to make four slight modifications. We need to point to the private keys for our Organization’s CA’s. You can locate these values in your crypto-config folder. For example, to locate the private key for Org1 we would follow this path - crypto-config/peerOrganizations/org1.example.com/ca/. The private key is a long hash value followed by _sk. The path for Org2 would be - crypto-config/peerOrganizations/org2.example.com/ca/.

In the docker-compose-e2e.yaml update the FABRIC_CA_SERVER_TLS_KEYFILE variable for ca0 and ca1. You also need to edit the path that is provided in the command to start the ca server. You are providing the same private key twice for each CA container.
Using CouchDB

The state database can be switched from the default (goleveldb) to CouchDB. The same chaincode functions are available with CouchDB, however, there is the added ability to perform rich and complex queries against the state database data content contingent upon the chaincode data being modeled as JSON.

To use CouchDB instead of the default database (goleveldb), follow the same procedures outlined earlier for generating the artifacts, except when starting the network pass docker-compose-couch.yaml as well:

CHANNEL_NAME=$CHANNEL_NAME TIMEOUT=<pick_a_value> docker-compose -f docker-compose-cli.yaml -f docker-compose-couch.yaml up -d

chaincode_example02 should now work using CouchDB underneath.

Note

If you choose to implement mapping of the fabric-couchdb container port to a host port, please make sure you are aware of the security implications. Mapping of the port in a development environment makes the CouchDB REST API available, and allows the visualization of the database via the CouchDB web interface (Fauxton). Production environments would likely refrain from implementing port mapping in order to restrict outside access to the CouchDB containers.

You can use chaincode_example02 chaincode against the CouchDB state database using the steps outlined above, however in order to exercise the CouchDB query capabilities you will need to use a chaincode that has data modeled as JSON, (e.g. marbles02). You can locate the marbles02 chaincode in the fabric/examples/chaincode/go directory.

We will follow the same process to create and join the channel as outlined in the Create & Join Channel section above. Once you have joined your peer(s) to the channel, use the following steps to interact with the marbles02 chaincode:

    Install and instantiate the chaincode on peer0.org1.example.com:

# be sure to modify the $CHANNEL_NAME variable accordingly for the instantiate command

peer chaincode install -n marbles -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles02
peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org0MSP.member','Org1MSP.member')"

    Create some marbles and move them around:

# be sure to modify the $CHANNEL_NAME variable accordingly

peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble1","blue","35","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble2","red","50","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["initMarble","marble3","blue","70","tom"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarble","marble2","jerry"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'
peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C $CHANNEL_NAME -n marbles -c '{"Args":["delete","marble1"]}'

    If you chose to map the CouchDB ports in docker-compose, you can now view the state database through the CouchDB web interface (Fauxton) by opening a browser and navigating to the following URL:

    http://localhost:5984/_utils

You should see a database named mychannel (or your unique channel name) and the documents inside it.

Note

For the below commands, be sure to update the $CHANNEL_NAME variable appropriately.

You can run regular queries from the CLI (e.g. reading marble2):

peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["readMarble","marble2"]}'

The output should display the details of marble2:

Query Result: {"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}

You can retrieve the history of a specific marble - e.g. marble1:

peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["getHistoryForMarble","marble1"]}'

The output should display the transactions on marble1:

Query Result: [{"TxId":"1c3d3caf124c89f91a4c0f353723ac736c58155325f02890adebaa15e16e6464", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}},{"TxId":"755d55c281889eaeebf405586f9e25d71d36eb3d35420af833a20a2f53a3eefd", "Value":{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"jerry"}},{"TxId":"819451032d813dde6247f85e56a89262555e04f14788ee33e28b232eef36d98f", "Value":}]

You can also perform rich queries on the data content, such as querying marble fields by owner jerry:

peer chaincode query -C $CHANNEL_NAME -n marbles -c '{"Args":["queryMarblesByOwner","jerry"]}'

The output should display the two marbles owned by jerry:

Query Result: [{"Key":"marble2", "Record":{"color":"red","docType":"marble","name":"marble2","owner":"jerry","size":50}},{"Key":"marble3", "Record":{"color":"blue","docType":"marble","name":"marble3","owner":"jerry","size":70}}]

A Note on Data Persistence

If data persistence is desired on the peer container or the CouchDB container, one option is to mount a directory in the docker-host into a relevant directory in the container. For example, you may add the following two lines in the peer container specification in the docker-compose-base.yaml file:

volumes:
 - /var/hyperledger/peer0:/var/hyperledger/production

For the CouchDB container, you may add the following two lines in the CouchDB container specification:

volumes:
 - /var/hyperledger/couchdb0:/opt/couchdb/data

Troubleshooting

    Always start your network fresh. Use the following command to remove artifacts, crypto, containers and chaincode images:

./byfn.sh -m down

    YOU WILL SEE ERRORS IF YOU DO NOT REMOVE CONTAINERS AND IMAGES
    If you see Docker errors, first check your version (should be 17.03.1 or above), and then try restarting your Docker process. Problems with Docker are oftentimes not immediately recognizable. For example, you may see errors resulting from an inability to access crypto material mounted within a container.
    If they persist remove your images and start from scratch:

docker rm -f $(docker ps -aq)
docker rmi -f $(docker images -q)

    If you see errors on your create, instantiate, invoke or query commands, make sure you have properly updated the channel name and chaincode name. There are placeholder values in the supplied sample commands.
    If you see the below error:

Error: Error endorsing chaincode: rpc error: code = 2 desc = Error installing chaincode code mycc:1.0(chaincode /var/hyperledger/production/chaincodes/mycc.1.0 exits)

You likely have chaincode images (e.g. dev-peer1.org2.example.com-mycc-1.0 or dev-peer0.org1.example.com-mycc-1.0) from prior runs. Remove them and try again.

docker rmi -f $(docker images | grep peer[0-9]-peer[0-9] | awk '{print $3}')

    If you see something similar to the following:

Error connecting: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure
Error: rpc error: code = 14 desc = grpc: RPC failed fast due to transport failure

Make sure you are running your network against the “1.0.0” images that have been retagged as “latest”.

If you see the below error:

[configtx/tool/localconfig] Load -> CRIT 002 Error reading configuration: Unsupported Config Type ""
panic: Error reading configuration: Unsupported Config Type ""

Then you did not set the FABRIC_CFG_PATH environment variable properly. The configtxgen tool needs this variable in order to locate the configtx.yaml. Go back and execute an export FABRIC_CFG_PATH=$PWD, then recreate your channel artifacts.

    To cleanup the network, use the down option:

./byfn.sh -m down

    If you see an error stating that you still have “active endpoints”, then prune your Docker networks. This will wipe your previous networks and start you with a fresh environment:

docker network prune

You will see the following message:

WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N]

Select y.

    If you continue to see errors, share your logs on the # fabric-questions channel on Hyperledger Rocket Chat.
