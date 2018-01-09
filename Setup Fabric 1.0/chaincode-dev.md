## 区块链应用开发
典型的区块链应用程序的工作过程如图13-1所示。其中，用户专注于与业务本身相关的应用程序；智能合约则封装了与区块链账本直接交互的相关过程，被应用程序调用。

![jpg](../images/blockchain-app.jpg)

## Chaincode接口
每个链码都需要实现以下Chaincode接口：

    type Chaincode interface {
    Init(stub ChaincodeStubInterface) pb.Response
    Invoke(stub ChaincodeStubInterface) pb.Response
    }

其中：

    ·Init：当链码收到实例化（instantiate）或升级（upgrade）类型的交易时，Init方法会被调用；
    ·Invoke：当链码收到调用（invoke）或查询（query）类型的交易时，Invoke方法会被调用。

## 链码结构

一个链码的必要结构如下所示：

    package main
    // 引入必要的包
    import (
    "github.com/hyperledger/fabric/core/chaincode/shim"
    pb "github.com/hyperledger/fabric/protos/peer"
    )
    // 声明一个结构体
    type SimpleChaincode struct {}
    // 为结构体添加Init方法
    func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
    // 在该方法中实现链码初始化或升级时的处理逻辑
    // 编写时可灵活使用stub中的API
    }
    // 为结构体添加 Invoke 方法
    func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
    // 在该方法中实现链码运行中被调用或查询时的处理逻辑
    // 编写时可灵活使用stub中的API
    }
    // 主函数，需要调用 shim.Start() 方法
    func main() {
    err := shim.Start(new(SimpleChaincode))
    if err != nil {
    fmt.Printf("Error starting Simple chaincode: %s", err)
    }
    }

1. 依赖包

从import代码段可以看到，链码需要引入如下的依赖包：

    ·"github.com/hyperledger/fabric/core/chaincode/shim"：shim包提供了链码与账本交互的中间层。链码通过shim.ChaincodeStub提供的方法来读取和修改账本状态；
    ·pb"github.com/hyperledger/fabric/protos/peer"：Init和Invoke方法需要返回pb.Response类型。

2. Init和Invoke方法
3. 
编写链码，关键是要实现Init和Invoke这两个方法。

当部署或升级链码时，Init方法会被调用。如同名字所描述的，该方法用来完成一些初始化的工作。当通过调用链码来做一些实际性的工作时，Invoke方法被调用，因此响应调用或查询的业务逻辑都需要在该方法中实现。

Init或Invoke方法以stub shim.ChaincodeStubInterface作为传入参数，pb.Response作为返回类型。其中，stub包含丰富的API，功能包括对账本进行操作、读取交易参数、调用其他链码等，链码开发者最好能够熟练使用这些API。

## 案例源码解析

