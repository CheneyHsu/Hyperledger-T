
# Chaincode for Operators
## Chaincode lifecycle

hyperledger Fabric API进行交互在blockchain网络各节点，the peers,orderer 和 MSPs允许一个包的形式，安装和升级的支持，实例化的节点链码。hyperledger fabric特定语言的SDK，hyperledger fabric API，方便应用程序的开发，虽然它可以被用来管理一个链码的周期。此外，hyperledger fabric API可以直接通过命令行访问。

我们提供四个命令来管理一个链码的生命周期：软件包，安装，实例化和升级。在未来的版本中，我们考虑将停止和启动交易禁用和重新启用chaincode，实际上不用去卸载它。在一个链码已成功安装和实例化，链码是活跃的（运行）和可交易的过程中通过调用事务。一个链码安装后可以任何时间升级。

## Packaging

链码包由3部分组成：:

* 链码，通过chaincodedeploymentspec或CDS定义。CDS定义码封装的代码和其他性质如名称和版本，
* 一个可选的实例化策略可用于背书和背书的政策描述的语法描述
*一套签名的实体，属于“自己”的链码的签名实体。

签名具有下列目的

* 建立一个所有权的链码
* 允许验证包的内容
* 允许检测包篡改.


## Creating the package

有2种包装链码，一个是单一链码多个使用者，因此码包需要有多重身份验证，需要最初的签名包，然后其他使用者连续签名。
 创建一个签名的链码包，使用下面的命令： 

    peer chaincode package -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -v 0 -s -S -i "AND('OrgA.admin')" ccpack.out

-s 选项创建多人签名
-S 该过程使用MSP在core.yaml的localmspoid属性值确定报的标识
-i 允许指定链码的policy，实例化政策具有相同的格式，支持政策和指定的身份可以实例化链码。在上面的例子中，只有组织管理允许实例化的链码。如果没有提供政策，默认使用的策略，只允许贵族的MSP管理员身份来实例化链码。

## Package signing

在创作时签署的，可交给其他业主查阅和签署。

 包含3个元素的signedcds： 

    CDS包含源代码、名称、版本、链码。

    的链码实例化政策，表示为支持政策。

    链码的业主名单，以背书方式定义。

如果未指定实例化策略，则默认策略为通道的任何MSP管理员

每个业主认可的chaincodedeploymentspec结合业主的身份（例如证书）和签约的综合结果。

chaincode所有者可以使用下面的命令以前创建的包签签：

    peer chaincode signpackage ccpack.out signedccpack.out

在`ccpack.out`和`signedccpack.out`分别是输入和输出的软件包，`signedccpack.out`包含在签约使用本地MSP附加签名。 

## Installing chaincode

安装事务包链码的源代码转换成规定的格式称为chaincodedeploymentspec（or CDS）并将其安装在一个对等节点将运行链码。

你必须在每个支持的渠道，将你的链码节点安装码。 

通过API安装了一个简单的`chaincodedeploymentspec` ，它将默认实例化政策，包括空的所有者列表。 

链码只能安装在支持拥有成员的链码节点上。无码的那些成员，不能执行码。但是，它们仍然可以验证并将交易提交给分类帐。

安装一个chaincode，发送`signedproposal`到`lifecycle system chaincode`（LSCC）`System Chaincode` 。例如，安装在部分简单的资产链码描述,使用CLI命令安装SACC码，像下面这样：

    peer chaincode install -n asset_mgmt -v 1.0 -p sacc

-p指定路径

为了在其他peer安装， `SignedProposal` 必须是MSP的管理员

## Instantiate

交易的实例化调用生命周期系统码（LSCC）创建并初始化一个信道码。这是一个链码信道绑定的过程：一个链码可以绑定到任何数量的通道，每个通道单独和独立操作。换句话说，不管有多少其他渠道，chaincode可以安装和实例化，状态保持隔离通道提交的事务。 

一个实例化的交易创造者必须满足的链码包括SignedCDS的实例化政策，也必须是该通道的一个写入者。这对于通道的安全，以防止恶意实体部署chaincodes或欺骗的成员在未绑定的渠道执行chaincodes是重要的。 

例如，记得默认实例化的政策是任何渠道MSP管理员，所以chaincode实例化交易创造者必须对渠道成员管理。当交易提案到成员，验证了交易者的签名与实例化policy。在将它提交给分类帐之前，在次期间进行事务验证操作。

实例化交易还建立在信道码的policy。签入policy描述了由通道成员接受的事务结果的认证要求。 

例如，使用CLI来实例化和初始化SACC码的状态给John和0，命令如下： 

    peer chaincode instantiate -n sacc -v 1.0 -c '{"Args":["john","0"]}' -P "OR ('Org1.member','Org2.member')"

Note

Note the endorsement policy (CLI uses polish notation), which requires an endorsement from either member of Org1 or Org2 for all transactions to sacc. That is, either Org1 or Org2 must sign the result of executing the Invoke on sacc for the transactions to be valid.

After being successfully instantiated, the chaincode enters the active state on the channel and is ready to process any transaction proposals of type ENDORSER_TRANSACTION. The transactions are processed concurrently as they arrive at the endorsing peer.
Upgrade

A chaincode may be upgraded any time by changing its version, which is part of the SignedCDS. Other parts, such as owners and instantiation policy are optional. However, the chaincode name must be the same; otherwise it would be considered as a totally different chaincode.

Prior to upgrade, the new version of the chaincode must be installed on the required endorsers. Upgrade is a transaction similar to the instantiate transaction, which binds the new version of the chaincode to the channel. Other channels bound to the old version of the chaincode still run with the old version. In other words, the upgrade transaction only affects one channel at a time, the channel to which the transaction is submitted.

Note

Note that since multiple versions of a chaincode may be active simultaneously, the upgrade process doesn’t automatically remove the old versions, so user must manage this for the time being.

There’s one subtle difference with the instantiate transaction: the upgrade transaction is checked against the current chaincode instantiation policy, not the new policy (if specified). This is to ensure that only existing members specified in the current instantiation policy may upgrade the chaincode.

Note

Note that during upgrade, the chaincode Init function is called to perform any data related updates or re-initialize it, so care must be taken to avoid resetting states when upgrading chaincode.
Stop and Start

Note that stop and start lifecycle transactions have not yet been implemented. However, you may stop a chaincode manually by removing the chaincode container and the SignedCDS package from each of the endorsers. This is done by deleting the chaincode’s container on each of the hosts or virtual machines on which the endorsing peer nodes are running, and then deleting the SignedCDS from each of the endorsing peer nodes:

Note

TODO - in order to delete the CDS from the peer node, you would need to enter the peer node’s container, first. We really need to provide a utility script that can do this.

docker rm -f <container id>
rm /var/hyperledger/production/chaincodes/<ccname>:<ccversion>

Stop would be useful in the workflow for doing upgrade in controlled manner, where a chaincode can be stopped on a channel on all peers before issuing an upgrade.
CLI

Note

We are assessing the need to distribute platform-specific binaries for the Hyperledger Fabric peer binary. For the time being, you can simply invoke the commands from within a running docker container.

To view the currently available CLI commands, execute the following command from within a running fabric-peer Docker container:

docker run -it hyperledger/fabric-peer bash
# peer chaincode --help

Which shows output similar to the example below:

Usage:
  peer chaincode [command]

Available Commands:
  install     Package the specified chaincode into a deployment spec and save it on the peer's path.
  instantiate Deploy the specified chaincode to the network.
  invoke      Invoke the specified chaincode.
  package     Package the specified chaincode into a deployment spec.
  query       Query using the specified chaincode.
  signpackage Sign the specified chaincode package
  upgrade     Upgrade chaincode.

Flags:
      --cafile string     Path to file containing PEM-encoded trusted certificate(s) for the ordering endpoint
  -C, --chainID string    The chain on which this command should be executed (default "testchainid")
  -c, --ctor string       Constructor message for the chaincode in JSON format (default "{}")
  -E, --escc string       The name of the endorsement system chaincode to be used for this chaincode
  -l, --lang string       Language the chaincode is written in (default "golang")
  -n, --name string       Name of the chaincode
  -o, --orderer string    Ordering service endpoint
  -p, --path string       Path to chaincode
  -P, --policy string     The endorsement policy associated to this chaincode
  -t, --tid string        Name of a custom ID generation algorithm (hashing and decoding) e.g. sha256base64
      --tls               Use TLS when communicating with the orderer endpoint
  -u, --username string   Username for chaincode operations when security is enabled
  -v, --version string    Version of the chaincode specified in install/instantiate/upgrade commands
  -V, --vscc string       The name of the verification system chaincode to be used for this chaincode

Global Flags:
      --logging-level string       Default logging level and overrides, see core.yaml for full syntax
      --test.coverprofile string   Done (default "coverage.cov")

Use "peer chaincode [command] --help" for more information about a command.

To facilitate its use in scripted applications, the peer command always produces a non-zero return code in the event of command failure.

Example of chaincode commands:

peer chaincode install -n mycc -v 0 -p path/to/my/chaincode/v0
peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a", "b", "c"]} -C mychannel
peer chaincode install -n mycc -v 1 -p path/to/my/chaincode/v1
peer chaincode upgrade -n mycc -v 1 -c '{"Args":["d", "e", "f"]} -C mychannel
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","e"]}'
peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C mychannel -n mycc -c '{"Args":["invoke","a","b","10"]}'

System chaincode

System chaincode has the same programming model except that it runs within the peer process rather than in an isolated container like normal chaincode. Therefore, system chaincode is built into the peer executable and doesn’t follow the same lifecycle described above. In particular, install, instantiate and upgrade do not apply to system chaincodes.

The purpose of system chaincode is to shortcut gRPC communication cost between peer and chaincode, and tradeoff the flexibility in management. For example, a system chaincode can only be upgraded with the peer binary. It must also register with a fixed set of parameters compiled in and doesn’t have endorsement policies or endorsement policy functionality.

System chaincode is used in Hyperledger Fabric to implement a number of system behaviors so that they can be replaced or modified as appropriate by a system integrator.

The current list of system chaincodes:

    LSCC Lifecycle system chaincode handles lifecycle requests described above.
    CSCC Configuration system chaincode handles channel configuration on the peer side.
    QSCC Query system chaincode provides ledger query APIs such as getting blocks and transactions.
    ESCC Endorsement system chaincode handles endorsement by signing the transaction proposal response.
    VSCC Validation system chaincode handles the transaction validation, including checking endorsement policy and multiversioning concurrency control.

Care must be taken when modifying or replacing these system chaincodes, especially LSCC, ESCC and VSCC since they are in the main transaction execution path. It is worth noting that as VSCC validates a block before committing it to the ledger, it is important that all peers in the channel compute the same validation to avoid ledger divergence (non-determinism). So special care is needed if VSCC is modified or replaced.
