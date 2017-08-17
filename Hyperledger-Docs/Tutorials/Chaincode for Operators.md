
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

There are two approaches to packaging chaincode. One for when you want to have multiple owners of a chaincode, and hence need to have the chaincode package signed by multiple identities. This workflow requires that we initially create a signed chaincode package (a SignedCDS) which is subsequently passed serially to each of the other owners for signing.

The simpler workflow is for when you are deploying a SignedCDS that has only the signature of the identity of the node that is issuing the install transaction.

We will address the more complex case first. However, you may skip ahead to the Installing chaincode section below if you do not need to worry about multiple owners just yet.

To create a signed chaincode package, use the following command:

peer chaincode package -n mycc -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -v 0 -s -S -i "AND('OrgA.admin')" ccpack.out

The -s option creates a package that can be signed by multiple owners as opposed to simply creating a raw CDS. When -s is specified, the -S option must also be specified if other owners are going to need to sign. Otherwise, the process will create a SignedCDS that includes only the instantiation policy in addition to the CDS.

The -S option directs the process to sign the package using the MSP identified by the value of the localMspid property in core.yaml.

The -S option is optional. However if a package is created without a signature, it cannot be signed by any other owner using the signpackage command.

The optional -i option allows one to specify an instantiation policy for the chaincode. The instantiation policy has the same format as an endorsement policy and specifies which identities can instantiate the chaincode. In the example above, only the admin of OrgA is allowed to instantiate the chaincode. If no policy is provided, the default policy is used, which only allows the admin identity of the peer’s MSP to instantiate chaincode.
Package signing

A chaincode package that was signed at creation can be handed over to other owners for inspection and signing. The workflow supports out-of-band signing of chaincode package.

The ChaincodeDeploymentSpec may be optionally be signed by the collective owners to create a SignedChaincodeDeploymentSpec (or SignedCDS). The SignedCDS contains 3 elements:

        The CDS contains the source code, the name, and version of the chaincode.
        An instantiation policy of the chaincode, expressed as endorsement policies.
        The list of chaincode owners, defined by means of Endorsement.

Note

Note that this endorsement policy is determined out-of-band to provide proper MSP principals when the chaincode is instantiated on some channels. If the instantiation policy is not specified, the default policy is any MSP administrator of the channel.

Each owner endorses the ChaincodeDeploymentSpec by combining it with that owner’s identity (e.g. certificate) and signing the combined result.

A chaincode owner can sign a previously created signed package using the following command:

peer chaincode signpackage ccpack.out signedccpack.out

Where ccpack.out and signedccpack.out are the input and output packages, respectively. signedccpack.out contains an additional signature over the package signed using the Local MSP.
Installing chaincode

The install transaction packages a chaincode’s source code into a prescribed format called a ChaincodeDeploymentSpec (or CDS) and installs it on a peer node that will run that chaincode.

Note

You must install the chaincode on each endorsing peer node of a channel that will run your chaincode.

When the install API is given simply a ChaincodeDeploymentSpec, it will default the instantiation policy and include an empty owner list.

Note

Chaincode should only be installed on endorsing peer nodes of the owning members of the chaincode to protect the confidentiality of the chaincode logic from other members on the network. Those members without the chaincode, can’t be the endorsers of the chaincode’s transactions; that is, they can’t execute the chaincode. However, they can still validate and commit the transactions to the ledger.

To install a chaincode, send a SignedProposal to the lifecycle system chaincode (LSCC) described in the System Chaincode section. For example, to install the sacc sample chaincode described in section Simple Asset Chaincode using the CLI, the command would look like the following:

peer chaincode install -n asset_mgmt -v 1.0 -p sacc

The CLI internally creates the SignedChaincodeDeploymentSpec for sacc and sends it to the local peer, which calls the Install method on the LSCC. The argument to the -p option specifies the path to the chaincode, which must be located within the source tree of the user’s GOPATH, e.g. $GOPATH/src/sacc. See the CLI section for a complete description of the command options.

Note that in order to install on a peer, the signature of the SignedProposal must be from 1 of the peer’s local MSP administrators.
Instantiate

The instantiate transaction invokes the lifecycle System Chaincode (LSCC) to create and initialize a chaincode on a channel. This is a chaincode-channel binding process: a chaincode may be bound to any number of channels and operate on each channel individually and independently. In other words, regardless of how many other channels on which a chaincode might be installed and instantiated, state is kept isolated to the channel to which a transaction is submitted.

The creator of an instantiate transaction must satisfy the instantiation policy of the chaincode included in SignedCDS and must also be a writer on the channel, which is configured as part of the channel creation. This is important for the security of the channel to prevent rogue entities from deploying chaincodes or tricking members to execute chaincodes on an unbound channel.

For example, recall that the default instantiation policy is any channel MSP administrator, so the creator of a chaincode instantiate transaction must be a member of the channel administrators. When the transaction proposal arrives at the endorser, it verifies the creator’s signature against the instantiation policy. This is done again during the transaction validation before committing it to the ledger.

The instantiate transaction also sets up the endorsement policy for that chaincode on the channel. The endorsement policy describes the attestation requirements for the transaction result to be accepted by members of the channel.

For example, using the CLI to instantiate the sacc chaincode and initialize the state with john and 0, the command would look like the following:

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
