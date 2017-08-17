
# Chaincode for Developers
## What is Chaincode?


chaincode主要处理业务逻辑通过该网络的成员同意，所以它类似于一个“智能合同”。Ledger状态的chaincode创造范围只对链码和不能直接访问另一个链码。给予相应的权限，chaincode可以调用另一个链码访问同一网络内的状态。 



在下面的章节中，我们将探讨链码通过链码开发应用程序。我们提出了一个简单的链码的示例应用程序和每个方法的目标在Shim API中。 

## Chaincode API

Every chaincode program must implement the Chaincode interface whose methods are called in response to received transactions. In particular the Init method is called when a chaincode receives an instantiate or upgrade transaction so that the chaincode may perform any necessary initialization, including initialization of application state. The Invoke method is called in response to receiving an invoke transaction to process transaction proposals.

The other interface in the chaincode “shim” APIs is the ChaincodeStubInterface which is used to access and modify the ledger, and to make invocations between chaincodes.

In this tutorial, we will demonstrate the use of these APIs by implementing a simple chaincode application that manages simple “assets”.


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

Initializing the Chaincode

Next, we’ll implement the Init function.

// Init is called during chaincode instantiation to initialize any data.
func (t *SimpleAsset) Init(stub shim.ChaincodeStubInterface) peer.Response {

}

Note

Note that chaincode upgrade also calls this function. When writing a chaincode that will upgrade an existing one, make sure to modify the Init function appropriately. In particular, provide an empty “Init” method if there’s no “migration” or nothing to be initialized as part of the upgrade.

Next, we’ll retrieve the arguments to the Init call using the ChaincodeStubInterface.GetStringArgs function and check for validity. In our case, we are expecting a key-value pair.

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

Next, now that we have established that the call is valid, we’ll store the initial state in the ledger. To do this, we will call ChaincodeStubInterface.PutState with the key and value passed in as the arguments. Assuming all went well, return a peer.Response object that indicates the initialization was a success.

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

Invoking the Chaincode

First, let’s add the Invoke function’s signature.

// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The 'set'
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {

}

As with the Init function above, we need to extract the arguments from the ChaincodeStubInterface. The Invoke function’s arguments will be the name of the chaincode application function to invoke. In our case, our application will simply have two functions: set and get, that allow the value of an asset to be set or its current state to be retrieved. We first call ChaincodeStubInterface.GetFunctionAndParameters to extract the function name and the parameters to that chaincode application function.

// Invoke is called per transaction on the chaincode. Each transaction is
// either a 'get' or a 'set' on the asset created by Init function. The Set
// method may create a new asset by specifying a new key-value pair.
func (t *SimpleAsset) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
    // Extract the function and args from the transaction proposal
    fn, args := stub.GetFunctionAndParameters()

}

Next, we’ll validate the function name as being either set or get, and invoke those chaincode application functions, returning an appropriate response via the shim.Success or shim.Error functions that will serialize the response into a gRPC protobuf message.

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

Implementing the Chaincode Application

As noted, our chaincode application implements two functions that can be invoked via the Invoke function. Let’s implement those functions now. Note that as we mentioned above, to access the ledger’s state, we will leverage the ChaincodeStubInterface.PutState and ChaincodeStubInterface.GetState functions of the chaincode shim API.

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

Pulling it All Together

Finally, we need to add the main function, which will call the shim.Start function. Here’s the whole chaincode program source.

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

Building Chaincode

Now let’s compile your chaincode.

go get -u --tags nopkcs11 github.com/hyperledger/fabric/core/chaincode/shim
go build --tags nopkcs11

Assuming there are no errors, now we can proceed to the next step, testing your chaincode.
Testing Using dev mode

Normally chaincodes are started and maintained by peer. However in “dev mode”, chaincode is built and started by the user. This mode is useful during chaincode development phase for rapid code/build/run/debug cycle turnaround.

We start “dev mode” by leveraging pre-generated orderer and channel artifacts for a sample dev network. As such, the user can immediately jump into the process of compiling chaincode and driving calls.
Install Hyperledger Fabric Samples

If you haven’t already done so, please install the Hyperledger Fabric Samples.

Navigate to the chaincode-docker-devmode directory of the fabric-samples clone:

cd chaincode-docker-devmode

Download Docker images

We need four Docker images in order for “dev mode” to run against the supplied docker compose script. If you installed the fabric-samples repo clone and followed the instructions to download-platform-specific-binaries, then you should have the necessary Docker images installed locally.

Note

If you choose to manually pull the images then you must retag them as latest.

Issue a docker images command to reveal your local Docker Registry. You should see something similar to following:

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

Note

If you retrieved the images through the download-platform-specific-binaries, then you will see additional images listed. However, we are only concerned with these four.

Now open three terminals and navigate to your chaincode-docker-devmode directory in each.
Terminal 1 - Start the network

docker-compose -f docker-compose-simple.yaml up

The above starts the network with the SingleSampleMSPSolo orderer profile and launches the peer in “dev mode”. It also launches two additional containers - one for the chaincode environment and a CLI to interact with the chaincode. The commands for create and join channel are embedded in the CLI container, so we can jump immediately to the chaincode calls.
Terminal 2 - Build & start the chaincode

docker exec -it chaincode bash

You should see the following:

root@d2629980e76b:/opt/gopath/src/chaincode#

Now, compile your chaincode:

cd sacc
go build

Now run the chaincode:

CORE_PEER_ADDRESS=peer:7051 CORE_CHAINCODE_ID_NAME=mycc:0 ./sacc

The chaincode is started with peer and chaincode logs indicating successful registration with the peer. Note that at this stage the chaincode is not associated with any channel. This is done in subsequent steps using the instantiate command.
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
