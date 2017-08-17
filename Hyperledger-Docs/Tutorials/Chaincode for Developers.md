
# Chaincode for Developers
## What is Chaincode?


chaincode主要处理业务逻辑通过该网络的成员同意，所以它类似于一个“智能合同”。Ledger状态的chaincode创造范围只对链码和不能直接访问另一个链码。给予相应的权限，chaincode可以调用另一个链码访问同一网络内的状态。 



在下面的章节中，我们将探讨链码通过链码开发应用程序。我们提出了一个简单的链码的示例应用程序和每个方法的目标在Shim API中。 

## Chaincode API

每码程序必须以实现接口的方法以响应接收到的交易。特别是init方法被调用时，chaincode接收实例化或升级交易使链码可以执行任何必要的初始化，包括初始化应用程序状态。调用方法响应于接收调用事务来处理事务。

在“shim”的`chaincodestubinterface API`是用于访问和修改总账的其他接口，调用之间chaincodes。


## Simple Asset Chaincode

我们的应用程序是一个基本的样品码创造资产（key-value pairs）的分类。 

## Choosing a Location for the Code

如果您还没有在Go中进行编程，您可能希望确保安装了Go编程语言并正确配置了系统。

现在，你会想作为一个$GOPATH/src/子目录中创建一个目录链码。

为了使事情简单，让我们使用以下命令： 

    mkdir -p $GOPATH/src/sacc && cd $GOPATH/src/sacc

现在，让我们创建源文件，我们进行编程： 

    touch sacc.go

## Housekeeping

首先，让我们从一些基础开始。作为每一个链码，它实现了在特定的链码接口、初始化和调用函数。所以，让我们为我们的链码添加必要的依赖包，import进来。我们将入口的shim包和protobuf包。

    package main

    import (
        "fmt"

        "github.com/hyperledger/fabric/core/chaincode/shim"
        "github.com/hyperledger/fabric/protos/peer"
    )

## Initializing the Chaincode

接下来，我们将实现init函数。 

    // Init is called during chaincode instantiation to initialize any data.
    func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {

    }

下一步，我们将检索参数使用`chaincodestubinterface.getstringargs`功能检查有效性的`init`。在我们的例子中，我们期望一个键值对。

        // Init is called during chaincode instantiation to initialize any
        // data. Note that chaincode upgrade also calls this function to reset
        // or to migrate data, so be careful to avoid a scenario where you
        // inadvertently clobber your ledger's data!
        func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
        // Get the args from the transaction proposal
        args := stub.GetStringArgs()
        if len(args) != 2 {
            return shim.Error("Incorrect arguments. Expecting a key and a value")
        }
        }


接下来，现在我们已经确定调用是有效的，我们将把初始状态存储在分类帐中。要做到这一点，需要调用`chaincodestubinterface.putstate`的键和值作为参数传递。假设一切正常，返回一个表示初始化成功的响应对象。 

    // Init is called during chaincode instantiation to initialize any
    // data. Note that chaincode upgrade also calls this function to reset
    // or to migrate data, so be careful to avoid a scenario where you
    // inadvertently clobber your ledger's data!
    func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
    // Get the args from the transaction proposal
    args := stub.GetStringArgs()
    if len(args) != 2 {
        return shim.Error("Incorrect arguments. Expecting a key and a value")
    }

    // Set up any variables or assets here by calling stub.PutState()

    // We store the key and the value on the ledger
    err := stub.PutState(args[0], []byte(args[1]))
    if err != nil {
        return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
    }
    return shim.Success(nil)
    }

## Invoking the Chaincode

首先，让我们添加调用函数的签名。

    // Invoke is called per transaction on the chaincode. Each transaction is
    // either a 'get' or a 'set' on the asset created by Init function. The 'set'
    // method may create a new asset by specifying a new key-value pair.
    func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {

    }

与上面的初始化函数，我们需要从`chaincodestubinterface`提取参数。调用函数的参数将被调用的函数名称码中的应用。在我们的例子中，我们的应用程序将有两个函数：`set和get`，允许设置资产的值或检索当前状态。我们首先提取函数名，函数的参数调用 `chaincodestubinterface.getfunctionandparameters`链码的应用。

    // Invoke is called per transaction on the chaincode. Each transaction is
    // either a 'get' or a 'set' on the asset created by Init function. The Set
    // method may create a new asset by specifying a new key-value pair.
    func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
        // Extract the function and args from the transaction proposal
        fn, args := stub.GetFunctionAndParameters()

    }

下一步，我们将验证函数名为set或get，并调用链码应用函数，返回一个shim.success或者shim.error。将序列化为消息响应函数GRPC protobuf。

    // Invoke is called per transaction on the chaincode. Each transaction is
    // either a 'get' or a 'set' on the asset created by Init function. The Set
    // method may create a new asset by specifying a new key-value pair.
    func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
        // Extract the function and args from the transaction proposal
        fn, args := stub.GetFunctionAndParameters()

        var result string
        var err error
        if fn == "set" {
                result, err = set(stub, args)
        } else {
                result, err = get(stub, args)
        }
        if err != nil {
                return shim.Error(err.Error())
        }

        // Return the result as success payload
        return shim.Success([]byte(result))
    }

## Implementing the Chaincode Application

值得指出的是，我们的链码的应用实现了两个功能，可以通过调用函数。现在让我们现在实现这些函数。值得注意的是，正如我们上面提到的，进入总账的状态，我们将利用`chaincodestubinterface.putstate`和`chaincodestubinterface.getstate`功能的chaincode shim API.

    // Set stores the asset (both key and value) on the ledger. If the key exists,
    // it will override the value with the new one
    func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
        if len(args) != 2 {
                return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
        }

        err := stub.PutState(args[0], []byte(args[1]))
        if err != nil {
                return "", fmt.Errorf("Failed to set asset: %s", args[0])
        }
        return args[1], nil
    }

    // Get returns the value of the specified asset key
    func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
        if len(args) != 1 {
                return "", fmt.Errorf("Incorrect arguments. Expecting a key")
        }

        value, err := stub.GetState(args[0])
        if err != nil {
                return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
        }
        if value == nil {
                return "", fmt.Errorf("Asset not found: %s", args[0])
        }
        return string(value), nil
    }

## Pulling it All Together

添加main函数

    package main

    import (
        "fmt"

        "github.com/hyperledger/fabric/core/chaincode/shim"
        "github.com/hyperledger/fabric/protos/peer"
    )

    // SimpleAsset implements a simple chaincode to manage an asset
    type SimpleAsset struct {
    }

    // Init is called during chaincode instantiation to initialize any
    // data. Note that chaincode upgrade also calls this function to reset
    // or to migrate data.
    func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {
        // Get the args from the transaction proposal
        args := stub.GetStringArgs()
        if len(args) != 2 {
                return shim.Error("Incorrect arguments. Expecting a key and a value")
        }

        // Set up any variables or assets here by calling stub.PutState()

        // We store the key and the value on the ledger
        err := stub.PutState(args[0], []byte(args[1]))
        if err != nil {
                return shim.Error(fmt.Sprintf("Failed to create asset: %s", args[0]))
        }
        return shim.Success(nil)
    }

    // Invoke is called per transaction on the chaincode. Each transaction is
    // either a 'get' or a 'set' on the asset created by Init function. The Set
    // method may create a new asset by specifying a new key-value pair.
    func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
        // Extract the function and args from the transaction proposal
        fn, args := stub.GetFunctionAndParameters()

        var result string
        var err error
        if fn == "set" {
                result, err = set(stub, args)
        } else { // assume 'get' even if fn is nil
                result, err = get(stub, args)
        }
        if err != nil {
                return shim.Error(err.Error())
        }

        // Return the result as success payload
        return shim.Success([]byte(result))
    }

    // Set stores the asset (both key and value) on the ledger. If the key exists,
    // it will override the value with the new one
    func set(stub shim.ChaincodeStubInterface, args []string) (string, error) {
        if len(args) != 2 {
                return "", fmt.Errorf("Incorrect arguments. Expecting a key and a value")
        }

        err := stub.PutState(args[0], []byte(args[1]))
        if err != nil {
                return "", fmt.Errorf("Failed to set asset: %s", args[0])
        }
        return args[1], nil
    }

    // Get returns the value of the specified asset key
    func get(stub shim.ChaincodeStubInterface, args []string) (string, error) {
        if len(args) != 1 {
                return "", fmt.Errorf("Incorrect arguments. Expecting a key")
        }

        value, err := stub.GetState(args[0])
        if err != nil {
                return "", fmt.Errorf("Failed to get asset: %s with error: %s", args[0], err)
        }
        if value == nil {
                return "", fmt.Errorf("Asset not found: %s", args[0])
        }
        return string(value), nil
    }

    // main function starts up the chaincode in the container during instantiate
    func main() {
        if err := shim.Start(new(SimpleAsset)); err != nil {
                fmt.Printf("Error starting SimpleAsset chaincode: %s", err)
        }
    }

## Building Chaincode

编译：

    go get -u --tags nopkcs11 github.com/hyperledger/fabric/core/chaincode/shim
    go build --tags nopkcs11

## Testing Using dev mode


通常在peer节点chaincodes启动和维护。然而在“开发模式”，链码的建立是由用户开始的，这种模式是有用的，在快速发展阶段，代码/码编译/运行/调试的循环周转。

我们进入“开发模式”，这样，用户可以立即跳转到编译码和驱动的调用过程。 


## Install Hyperledger Fabric Samples

下载fabric-samples，进入 chaincode-docker-devmode 目录:

    cd chaincode-docker-devmode

## Download Docker images

我们所需的docker镜像.

    docker images
    REPOSITORY                     TAG                                  IMAGE ID            CREATED             SIZE
    hyperledger/fabric-tools       latest                               e09f38f8928d        4 hours ago         1.32 GB
    hyperledger/fabric-tools       x86_64-1.0.0                         e09f38f8928d        4 hours ago         1.32 GB
    hyperledger/fabric-orderer     latest                               0df93ba35a25        4 hours ago         179 MB
    hyperledger/fabric-orderer     x86_64-1.0.0                         0df93ba35a25        4 hours ago         179 MB
    hyperledger/fabric-peer        latest                               533aec3f5a01        4 hours ago         182 MB
    hyperledger/fabric-peer        x86_64-1.0.0                         533aec3f5a01        4 hours ago         182 MB
    hyperledger/fabric-ccenv       latest                               4b70698a71d3        4 hours ago         1.29 GB
    hyperledger/fabric-ccenv       x86_64-1.0.0                         4b70698a71d3        4 hours ago         1.29 GB


## Terminal 1 - Start the network

    docker-compose -f docker-compose-simple.yaml up


## Terminal 2 - Build & start the chaincode

    docker exec -it chaincode bash

You should see the following:

    root@d2629980e76b:/opt/gopath/src/chaincode#

Now, compile your chaincode:

    cd sacc
    go build

Now run the chaincode:

    CORE_PEER_ADDRESS=peer:7051 CORE_CHAINCODE_ID_NAME=mycc:0 ./sacc

Terminal 3 - Use the chaincode

Even though you are in --peer-chaincodedev mode, you still have to install the chaincode so the life-cycle system chaincode can go through its checks normally. This requirement may be removed in future when in --peer-chaincodedev mode.

We’ll leverage the CLI container to drive these calls.

    docker exec -it cli bash

    peer chaincode install -p chaincodedev/chaincode/sacc -n mycc -v 0
    peer chaincode instantiate -n mycc -v 0 -c '{"Args":["a","10"]}' -C myc

Now issue an invoke to change the value of “a” to “20”.

    peer chaincode invoke -n mycc -c '{"Args":["set", "a", "20"]}' -C myc

Finally, query a. We should see a value of 20.

    peer chaincode query -n mycc -c '{"Args":["query","a"]}' -C myc

Testing new chaincode

By default, we mount only sacc. However, you can easily test different chaincodes by adding them to the chaincode subdirectory and relaunching your network. At this point they will be accessible in your chaincode container.
