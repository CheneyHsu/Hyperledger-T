
# Chaincode Tutorials
## What is Chaincode?


Chaincode is a program, written in Go, and eventually in other programming languages such as Java, that implements a prescribed interface. Chaincode runs in a secured Docker container isolated from the endorsing peer process. Chaincode initializes and manages ledger state through transactions submitted by applications.

A chaincode typically handles business logic agreed to by members of the network, so it may be considered as a “smart contract”. State created by a chaincode is scoped exclusively to that chaincode and can’t be accessed directly by another chaincode. However, within the same network, given the appropriate permission a chaincode may invoke another chaincode to access its state.

链码是一种程序，go编写，并最终在其他编程语言如java，实现指定接口。链码是从支持同行过程中分离获得Docker容器。链码初始化并通过提交应用交易管理台账状态。

chaincode主要处理业务逻辑同意通过该网络的成员，因此它可以被认为是一个“聪明的合同”。采用链码创造状态是范围只对链码和不能直接访问另一个链码。然而，在同一网络中，给予相应的权限可以调用另一个链码码来访问它的状态。 

## Two Personas



我们提供两个不同的角度对链码。一、从一个开发区块应用/解决方案开发商有权码应用开发者的角度，和其他面向blockchain网络运营商谁是负责管理一个blockchain网络运营商链码，利用hyperledger API安装，实例化，并升级码，参与一个链码的开发应用。 

We offer two different perspectives on chaincode. One, from the perspective of an application developer developing a blockchain application/solution entitled Chaincode for Developers, and the other, Chaincode for Operators oriented to the blockchain network operator who is responsible for managing a blockchain network, and who would leverage the Hyperledger Fabric API to install, instantiate, and upgrade chaincode, but would likely not be involved in the development of a chaincode application.
