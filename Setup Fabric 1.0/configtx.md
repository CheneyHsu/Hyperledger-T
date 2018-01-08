## configtx
* 由于区块链系统自身的分布式特性，对其中配置进行更新和管理是一件很有挑战的任务。一旦出现不同节点之间配置不一致，就可能导致整个网络功能异常。
* 在Fabric网络中，通过采用配置交易（Configuration Transaction，ConfigTX）这一创新设计来实现对通道相关配置的更新。配置更新操作如果被执行，也要像应用交易一样经过网络中节点的共识确认。
* configtxgen（Configuration Transaction Generator）工具是一个很重要的辅助工具，它可以配合cryptogen生成的组织结构身份文件使用，离线生成跟通道有关的配置信息，相关的实现在common/configtx包下。
* 主要功能有如下三个：

        · 生成启动Orderer需要的初始区块，并支持检查区块内容；
        · 生成创建应用通道需要的配置交易，并支持检查交易内容；
        · 生成锚点Peer的更新配置交易。

* 默认情况下，configtxgen工具会依次尝试从$FABRIC_CFG_PATH环境变量指定的路径，当前路径和/etc/hyperledger/fabric路径下查找configtx.yaml配置文件并读入，作为默认的配置。环境变量中以CONFIGTX_前缀开头的变量也会被作为配置项。
* Fabric代码中也提供了一些示例配置文件可以作为参考。
* configtx.yaml配置文件一般包括四个部分：Profiles、Organizations、Orderer和Application。

        ·Profiles：一系列通道配置模板，包括Orderer系统通道模板和应用通道类型模板；
        ·Organizations：一系列组织结构定义，被其他部分引用；
        ·Orderer：Orderer系统通道相关配置，包括Orderer服务配置和参与Ordering服务的可用组织信息；
        ·Application：应用通道相关配置，主要包括参与应用网络的可用组织信息。

#### Profiles
* 定义了一系列的Profile，每个Profile代表了某种应用场景下的通道配置模板，包括Orderer系统通道模板或应用通道模板，有时候也混合放到一起。
* Orderer系统通道模板必须包括Orderer、Consortiums信息：
* Orderer：指定Orderer系统通道自身的配置信息。包括Ordering服务配置（包括类型、地址、批处理限制、Kafka信息、最大应用通道数目等），参与到此Orderer的组织信息。网络启动时，必须首先创建Orderer系统通道；
* Consortiums：Orderer所服务的联盟列表。每个联盟中组织彼此使用相同的通道创建策略，可以彼此创建应用通道。
* 应用通道模板中必须包括Application、Consortium信息：
 
        ·Application：指定属于某应用通道的信息，主要包括属于通道的组织信息；
        ·Consortium：该应用通道所关联联盟的名称。
>一般建议将Profile分为Orderer系统通道配置（至少包括指定Orderers和Consortiums）和应用通道配置（至少包括指定Applications和Consortium）两种，分别进行编写，如下所示：
##### Profiles:
```
TwoOrgsOrdererGenesis: # Orderer系统通道配置。通道为默认配置，添加一个OrdererOrg
组织；联盟为默认的SampleConsortium联盟，添加了两个组织
Orderer:
<<: *OrdererDefaults
Organizations: # 属于Orderer通道的组织
- *OrdererOrg
Consortiums:
SampleConsortium: # 创建更多应用通道时的联盟
Organizations:
- *Org1
- *Org2
TwoOrgsChannel: # 应用通道配置。默认配置的应用通道，添加了两个组织。联盟为SampleConsortium
Application:
<<: *ApplicationDefaults
Organizations: # 初始加入应用通道的组织
- *Org1
- *Org2
Consortium: SampleConsortium # 联盟
代码中代表一个Profile相关配置的数据结构定义如下，同时支持Orderer系统通道和应用通道的配置数据：
type Profile struct {
// 应用通道相关
Consortium  string                 `yaml:"Consortium"`
Application *Application           `yaml:"Application"`
// Orderer系统通道相关
Orderer     *Orderer               `yaml:"Orderer"`
Consortiums map[string]*Consortium `yaml:"Consortiums"`
}
```
>提示　在YAML文件中，&KEY所定位的字段信息，可以通过'<<：KEY'语法来引用，相当于导入定位部分的内容。
##### Organizations
* 这一部分主要定义一系列的组织结构，根据服务对象类型的不同，包括Orderer组织和普通的应用组织。
* Orderer类型组织包括名称、ID、MSP文件路径、管理员策略等，应用类型组织还会配置锚点Peer信息。这些组织都会被Profiles部分引用使用。
>配置文件内容如下所示：
```
Organizations:
- &OrdererOrg
Name: OrdererOrg
ID: OrdererOrg # MSP的ID
MSPDir: msp # MSP相关文件所在本地路径
AdminPrincipal: Role.ADMIN # 组织管理员所需要的身份，可以为Role.ADMIN或
Role.MEMBER
- &Org1
Name: Org1MSP
ID: Org1MSP # MSP的ID
MSPDir: msp # MSP相关文件所在本地路径
AnchorPeers: # 锚节点地址，用于跨组织的Gossip通信
- Host: peer0.org1.example.com
Port: 7051
- &Org2
Name: Org2MSP
ID: Org2MSP # MSP的ID
MSPDir: msp # MSP相关文件所在本地路径
AnchorPeers: # 锚节点地址，用于跨组织的Gossip通信
- Host: peer0.org2.example.com
Port: 7051
代表一个组织的相关配置的数据结构定义如下所示，包括名称、ID、MSP文件目录、管理员身份规则、锚节点等：
type Organization struct {
Name           string `yaml:"Name"`
ID             string `yaml:"ID"`
MSPDir         string `yaml:"MSPDir"`
AdminPrincipal string `yaml:"AdminPrincipal"`
AnchorPeers []*AnchorPeer `yaml:"AnchorPeers"`
}
```
##### 3.Orderer
* 示例排序节点的配置，默认是solo类型，不包括任何组织。源码中自带的配置文件内容如下所示，包括类型、地址、批处理超时和字节数限制、最大通道数、参与组织等。被Profiles部分引用：
```
Orderer: &OrdererDefaults
OrdererType: solo # orderer类型，包括solo（单点）和kafka（kafka集群作为
后端）
Addresses: # 服务地址
- orderer.example.com:7050
BatchTimeout: 2s # 创建批量交易的最大超时，一批交易可以构建一个区块
BatchSize: # 控制写入到区块内交易的个数
MaxMessageCount: 10 # 一批消息最大个数
AbsoluteMaxBytes: 98 MB # batch最大字节数，任何时候不能超过
PreferredMaxBytes: 512 KB # 通常情况下，batch建议字节数；极端情况下，如单个消息就超过该值（但未超过最大限制），仍允许构成区块
MaxChannels: 0 # ordering服务最大支持的应用通道数，默认为0，表示无限制
Kafka:
Brokers: # Kafka brokers作为orderer后端
- 127.0.0.1:9092
Organizations: # 参与维护orderer的组织，默认为空
代表Orderer相关配置的数据结构定义如下所示，包括类型、地址、块超时和限制、Kafka信息，支持的最大通道数、关联的组织信息等：
type Orderer struct {
OrdererType   string          `yaml:"OrdererType"`
Addresses     []string        `yaml:"Addresses"`
BatchTimeout  time.Duration   `yaml:"BatchTimeout"`
BatchSize     BatchSize       `yaml:"BatchSize"`
Kafka         Kafka           `yaml:"Kafka"`
MaxChannels   uint64          `yaml:"MaxChannels"`
Organizations []*Organization `yaml:"Organizations"`
}
```
##### Application
* 示例应用通道相关的信息，不包括任何组织。被Profiles部分引用。源码中自带的配置文件内容如下所示：
```
Application: &ApplicationDefaults
Organizations: # 加入到通道中的组织的信息
代表Application相关配置的数据结构定义如下所示，记录所关联的组织：
type Application struct {
Organizations []*Organization `yaml:"Organizations"`
}
```