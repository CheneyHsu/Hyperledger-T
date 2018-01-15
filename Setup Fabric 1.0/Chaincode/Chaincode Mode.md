### ChainCode
* 链上代码（chaincode），简称链码，一般是指用户编写的应用代码。
* 链码被部署在Fabric网络节点上，运行在隔离沙盒（目前为Docker容器）中，并通过gRPC协议与相应的Peer节点进行交互，以操作分布式账本中的数据。
* 启动Fabric网络后，可以通过命令行或SDK进行链码操作，验证网络运行是否正常。
* 用户链码有别于系统链码（System Chaincode）。系统链码指的是Fabric Peer中负责系统配置、背书、验证等平台功能的逻辑，运行在Peer进程内.

1. 链码操作命令
* 用户可以通过命令行方式操作链码，支持的链码子命令包括install、instantiate、invoke、query、upgrade、package、signpackage等，未来还会支持start、stop命令。大部分命令（除了package、signpackage外）的处理过程都是类似的，创建签名提案消息，发给Peer进行背书，获取ProposalResponse消息。
* 特别是，instantiate、upgrade、invoke等子命令还需要根据ProposalResponse消息创建SignedTX，发送给Orderer进行排序和广播全网执行。package、signpackage子命令作为本地操作，无需与Peer或Orderer打交道。
![jpg](../images/chaincode-1.jpg)

2. 命令参数
![jpg](../images/chaincode-2.jpg)
![jpg](../images/chaincode-3.jpg)
![jpg](../images/chaincode-4.jpg)

3. 安装链码
* install命令将链码的源码和环境等内容封装为一个链码安装打包文件（Chaincode Install Package，CIP），并传输到背书节点。背书节点解析后一般会保存在$CORE_PEER_FILESYSTEMPATH/chaincodes/目录下。安装链码只需要与Peer打交道。
* 打包文件以name.version命名，主要包括如下内容：
* ·ChaincodeDeploymentSpec：链码的源码和一些关联环境，如名称和版本；
* ·链码实例化策略，默认是任意通道上的MSP管理员身份均可；
* ·拥有这个链码的实体的证书和签名；
* ·安装时，本地MSP管理员的签名。
* ChaincodeDeploymentSpec（CDS）结构包括了最核心的ChaincodeSpec（CS）数据结构，同时也被其他链码命令（如实例化命令和升级命令）使用，如图所示。
![jpg](../images/chaincode-5.jpg)
* 例如，采用如下命令会部署test_cc.1.0的打包部署文件到背书节点：
```
$ peer chaincode install \
-n test_cc \
-v 1.0 \
-p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
```
* 链码安装实现整体流程如图所示:
![jpg](../images/chaincode-6.jpg)
* 主要步骤包括:

    1）首先是构造签名提案消息（SignedProposal）。
        
        a）调用InitCmdFactory（isEndorserRequired，isOrdererRequired bool）（*ChaincodeCmdFactory，error）方法，初始化EndoserClient、Signer等结构。这一步初始化操作对于所有链码子命令来说都是类似的，会初始化不同的结构。
        b）然后根据命令行参数进行解析，判断是根据传入的打包文件直接读取ChaincodeDeploymentSpec（CDS）结构，还是根据传入参数从本地链码文件来重新构造。
        c）以本地重新构造情况为例，首先根据命令行中传入的路径、名称等信息，构造生成ChaincodeSpec（CS）结构。
        d）利用ChaincodeSpec结构，结合链码包数据生成一个ChaincodeDeploymentSpec结构（chainID为空），调用本地的install（msg proto.Message，cf*ChaincodeCmdFactory）error方法。
        e）install方法基于传入的ChaincodeDeploymentSpec结构，构造一个对生命周周期管理系统链码（LSCC）调用的ChaincodeSpec结构，其中，Type为ChaincodeSpec_GOLANG，ChaincodeId.Name为“lscc”，Input为“install”+ChaincodeDeploymentSpec。进一步地，构造了一个LSCC的ChaincodeInvocationSpec（CIS）结构，对ChaincodeSpec结构进行封装。
        f）基于LSCC的ChaincodeInvocationSpec结构，添加头部结构，生成一个提案（Proposal）结构。其中，通道头部中类型为ENDORSER_TRANSACTION，TxID为对随机数+签名实体，进行Hash。
        g）对Proposal进行签名，转化为一个签名后的提案消息SignedProposal。

    2）通过EndorserClient经由gRPC通道发送给Peer的ProcessProposal（ctx context.Context，in*SignedProposal，opts...grpc.CallOption）（*ProposalResponse，error）接口。

    3）Peer模拟运行生命周期链码的调用交易进行处理，检查格式、签名和权限等，通过则保存到本地文件系统。
* 图给出了链码安装过程中所涉及的数据结构，这些数据结构对于大部分链码操作命令都是类似的，其中最重要的是ChannelHeader结构和ChaincodeSpec结构中参数的差异。
![jpg](../images/chaincode-7.jpg)

4. 实例化链码
* instantiate命令通过构造生命周期管理系统链码（Lifecycle System Chaincode，LSCC）的交易，将安装过的链码在指定通道上进行实例化调用，在节点上创建容器启动，并执行初始化操作。实例化链码需要同时跟Peer和Orderer打交道。
* 执行instantiate命令的用户身份必须满足实例化的策略，并且在所指定的通道上拥有写（Write）权限。在instantiate命令中可以通过“-P”参数指定链码的背书策略（Endorsement Policy），不满足背书策略的链码调用将在Commit阶段被作废。
* 例如，如下命令会启动test_cc.1.0链码，会将参数'{"Args"：["init"，"a"，"100"，"b"，"200"]}'传入链码中的Init（）方法执行。命令会生成一笔交易，因此需指定排序节点地址：
```
$ CHANNEL_NAME="businesschannel"
$ peer chaincode instantiate \
-o orderer0:7050 \
-C businesschannel \
-n test_cc \
-v 1.0 \
-C ${CHANNEL_NAME} \
-c '{"Args":["init","a","100","b","200"]}' \
-P "OR ('Org1MSP.member','Org2MSP.member')"
```
* 链码实例化整体流程
![jpg](../images/chaincode-8.jpg)
* 主要步骤包括：

    1）首先，类似链码安装命令，需要创建一个SignedProposal消息。注意instantiate和upgrade支持policy、escc、vscc等参数。LSCC的ChaincodeSpec结构中，Input中包括类型（“deploy”）、通道ID、ChaincodeDeploymentSpec结构、背书策略、escc和vscc等。

    2）调用EndorserClient，发送gRPC消息，将签名后的Proposal发给指定的Peer节点（Endorser），调用ProcessProposal（ctx context.Context，in*SignedProposal，opts...grpc.CallOption）（*ProposalResponse，error）方法，进行背书处理。节点会模拟运行LSCC的调用交易，启动链码容器。实例化成功后会返回ProposalResponse消息（其中包括背书签名）。

    3）根据Peer返回的ProposalResponse消息，创建一个SignedTX（Envelop结构的交易，带有签名）。

    4）使用BroadcastClient将交易消息通过gRPC通道发给Orderer，Orderer会进行全网排序，并广播给Peer进行确认提交。

* 其中，SignedProposal结构如图所示。
![jpg](../images/chaincode-9.jpg)
* 交易Envelope结构如图所示:
![jpg](../images/chaincode-10.jpg)
* Peer 返回的ProposalResponese消息定义如下:
```
type ProposalResponse struct {
// 消息协议版本
Version int32 `protobuf:"varint,1,opt,name=version" json:"version,
omitempty"`
// 消息创建时的时间戳
Timestamp *google_protobuf1.Timestamp `protobuf:"bytes,2,opt,name=tim
estamp" json:"timestamp,omitempty"`
// 返回消息，包括状态、消息、元数据载荷等
Response *Response `protobuf:"bytes,4,opt,name=response" json:
"response,omitempty"`
// 数据载荷，包括提案的 Hash 值，和扩展的行动等
Payload []byte `protobuf:"bytes,5,opt,name=payload,proto3" json:
"payload,omitempty"`
// 背书信息列表，包括背书者的证书，以及其对“载荷+背书者证书”的签名
Endorsement *Endorsement `protobuf:"bytes,6,opt,name=endorsement" json:
"endorsement,omitempty"`
}
```

4. 调用链码

* 通过invoke命令可以调用运行中的链码的方法。“-c”参数指定的函数名和参数会被传入到链码的Invoke（）方法进行处理。调用链码操作需要同时跟Peer和Orderer打交道。
* 例如，对部署成功的链码执行调用操作，由a向b转账10元。在peer0容器中执行如下操作，注意验证最终结果状态正常response：<status：200 message："OK">：

```
$ peer chaincode invoke \
-o orderer0:7050 \
-n test_cc \
-C ${CHANNEL_NAME} \
-c '{"Args":["invoke","a","b","10"]}'
```

这一命令会调用最新版本的test_cc链码，将参数'{"Args"：["invoke"，"a"，"b"，"10"]}'传入链码中的Invoke（）方法执行。命令会生成一笔交易，需指定排序者地址。

需要注意，invoke命令不支持指定链码版本，只能调用最新版本的链码。
