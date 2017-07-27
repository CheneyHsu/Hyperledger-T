# Setup E2E CouchDB

## 解析
fabric 1.0 中有3种类型的数据存储，一种是文件系统方式的区块链数据（类似比特币），在Fabric 1.0中的区块链存储了Transaction订单读写集，而这个读写集读什么？写什么？其实就是State Database，也叫作World State。以键值对的形式存储了Chaincode中操作的业务数据。另外还有就是对历史数据和区块链索引的数据库。

![png](../images/CouchDB.png)
> The Pic Form Network

区块链是文件系统，目前不支持更改，历史数据和区块链的索引是LevelDB，也不支持更改。而State Database是直接和业务相关的，所以提供了替换数据库，当前仅支持LevelDB和可选择的CouchDB。

CouchDB不一定是最优秀的，只是fabric相对开发资源有限，1.0的时候仅选择了CouchDB，其实很多人也考也Mysql或者MongoDB。

### 部署规划
CouchDB是一个完全局域RESTful API的键值数据库，可以不需要任何客户端，只需要通过HTTP请求就可以操作数据库了。LevelDB是Peer的本地数据库，那么肯定是和Peer一对一的关系，那么CouchDB是个网络数据库，应该和Peer是什么样一个关系呢？在生产环境中，我们会为每个组织部署节点，而且为了高可用，可能会在一个组织中部署多个Peer。同样我们在一个组织中也部署多个CouchDB，每个Peer对应一个CouchDB。

### 必要条件
完整的E2E环境部署后并没有包含CouchDB，需要在使用脚本启动的时候添加couchdb参数。

    1:停止已经启动的E2E环境
    ./network_setup.sh down
    2:启动CouchDB
    ./network_setup.sh up mychannel 60 couchdb

### 测试访问CouchDB
通过浏览器访问http://192.168.56.101:5984/_utils
<IP地址部分更换成对应的主机IP>
![png](../images/CouchDB-1.png)
点击mychannel数据库->Run A Query with Mango,可以进行数据查询

### CouchDB 直接查询
查看mychannel下所有信息:

    curl http://192.168.56.101:5984/mychannel/_all_docs

State DataBase 返回如下：

    {"total_rows":4,"offset":0,"rows":[
    {"id":"lscc\u0000mycc","key":"lscc\u0000mycc","value":{"rev":"1-2e7d10ba4fe6c88fee76dd0da50e3dc4"}},
    {"id":"mycc\u0000a","key":"mycc\u0000a","value":{"rev":"2-2af72e502c2b43c73064728852103fbf"}},
    {"id":"mycc\u0000b","key":"mycc\u0000b","value":{"rev":"2-9c9d6a611441e331be96e2d507e36265"}},
    {"id":"statedb_savepoint","key":"statedb_savepoint","value":{"rev":"5-1a3bfd954a062e55fcf34109c1d18acf"}}
    ]}

  查询其中的一条数据，只需要用/ChannelId/id来查询，比如查询：statedb_savepoint

    curl http://192.168.56.101:5984/mychannel/statedb_savepoint
statedb_savepoint 返回如下：

    {"_id":"statedb_savepoint","_rev":"5-1a3bfd954a062e55fcf34109c1d18acf","BlockNum":4,"TxNum":0,"UpdateSeq":"9-g1AAAAEzeJzLYWBg4MhgTmHgzcvPy09JdcjLz8gvLskBCjMlMiTJ____PyuRAYeCJAUgmWQPVsOES40DSE08WA0jLjUJIDX1eO3KYwGSDA1ACqhsPiF1CyDq9uN2F0TdAYi6-4TMewBRB3QfSxYAi9hi_w"}

查询业务数据是“ChainCodeName\u0000数据：

    curl http://192.168.56.101:5984/mychannel/mycc\u0000a
返回如下：（查找不到，因为0000要进行格式转换）

    {"error":"not_found","reason":"missing"}
正确查询如下：（把\u0000替换为%00）

    curl http://192.168.56.101:5984/mychannel/mycc%00a
返回信息如下：

    {"_id":"mycc\u0000a","_rev":"2-2af72e502c2b43c73064728852103fbf","chaincodeid":"mycc","version":"4:0","_attachments":{"valueBytes":{"content_type":"application/octet-stream","revpos":2,"digest":"md5-qpvq4/JGMCgu7WtvFu5zbg==","length":2,"stub":true}}}

## 部署新链进行测试
虽然区块链是一个只能插入和查询的数据库，但是业务数据是存放在State Database中的，如果直接修改了CouchDB的数据，那么接下来的查询和事务是直接基于修改后的CouchDB的，并不会去检查区块链中的记录，所以理论上是可以通过直接改CouchDB来实现业务数据的修改。

以官方的Marble为例，看看修改CouchDB会怎么样？
###### Install，instantiate和初始化数据：

    1：docker exec -it cli bash

    2：peer chaincode install -o orderer.example.com:7050 -n marbles02 -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/marbles02

    3：peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C mychannel -n marbles02  -v 1.0  -c '{"Args":["init"]}' -P "OR        ('Org1MSP.member','Org2MSP.member')"

    4：peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C mychannel -n marbles02  -v 1.0 -c  ' {"Args":["initMarble","marble01","red","1000","tom"]}'

    5：peer chaincode query -C mychannel -n marbles02 -c  '{"Args":["readMarble","marble01"]}'
      返回结果如下：
      Query Result: {"color":"red","docType":"marble","name":"marble01","owner":"tom","size":1000}

###### 通过curl直接查询CouchDB中的数据：

    curl http://192.168.56.101:5984/mychannel/marbles02%00marble2

返回结果如下：

    {"_id":"marbles02\u0000marble01","_rev":"1-936b03f2b57343d8376beddb48335c64","chaincodeid":"marbles02","data":{"docType":"marble","name":"marble01","color":"red","size":1000,"owner":"tom"},"version":"20:0"}

查看历史：

    peer chaincode query -C mychannel -n marbles02 -c '{"Args":["getHistoryForMarble","marble01"]}'
    返回1条记录：
    "Value":{"docType":"marble","name":"marble01","color":"red","size":1000,"owner":"tom"}, "Timestamp":"2017-07-19 08:56:22.754146148 +0000 UTC", "IsDelete":"false"}]
直接查询数据库：

    curl http://192.168.56.101:5984/mychannel/marbles02%00marble01
    返回如下：
    {"_id":"marbles02\u0000marble01","_rev":"1-936b03f2b57343d8376beddb48335c64","chaincodeid":"marbles02","data":{"docType":"marble","name":"marble01","color":"red","size":1000,"owner":"tom"},"version":"20:0"}

###### 修改数据的内容
修改数据库把red改为black，1000改成9999

      curl -X PUT http://192.168.56.101:5984/mychannel/marbles02%00marble01 -d '{"_id":"marbles02\u0000marble2","_rev":"1-936b03f2b57343d8376beddb48335c64","chaincodeid":"marbles02","data":{"docType":"marble","name":"marble01","color":"green","size":9999,"owner":"tom"},"version":"6:0"}'
      返回结果如下：
      {"ok":true,"id":"marbles02\u0000marble01","rev":"2-883288d192fa9c1ec70fc17d733a0e28"}

再次查看历史（依然只有一条记录，并没有我们修改的记录，数据也没有变化）

    "Value":{"docType":"marble","name":"marble01","color":"red","size":1000,"owner":"tom"}, "Timestamp":"2017-07-19 08:56:22.754146148 +0000 UTC", "IsDelete":"false"}]
直接查询数据库

    curl http://192.168.56.101:5984/mychannel/marbles02%00marble01
    结果返回：（数据库内容已经变化）
    {"_id":"marbles02\u0000marble01","_rev":"2-883288d192fa9c1ec70fc17d733a0e28","chaincodeid":"marbles02","data":{"docType":"marble","name":"marble01","color":"green","size":9999,"owner":"tom"},"version":"6:0"}

peer查询
    peer chaincode query -C mychannel -n marbles02 -c '{"Args":["readMarble","marble02"]}'
    返回结果：（链和数据库同步）
    Query Result: {"color":"green","docType":"marble","name":"marble01","owner":"tom","size":9999}

###### 进行Transaction试试（转账交易）

    peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marbles02 -c '{"Args":["transferMarble","marble01","jerry"]}'
    系统返回：（成功）
    2017-07-19 09:07:15.379 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 00a Chaincode invoke successful. result: status:200

再次查询（已经发生改变，交易成功）

    peer chaincode query -C mychannel -n marbles02 -c '{"Args":["readMarble","marble01"]}'
    返回结果
    Query Result: {"color":"green","docType":"marble","name":"marble01","owner":"jerry","size":9999}

直接查询数据库（交易已经记录并生效）

    {"_id":"marbles02\u0000marble01","_rev":"3-7d01e3c98d30989c7b4353866720b2d0","chaincodeid":"marbles02","data":{"docType":"marble","name":"marble01","color":"green","size":9999,"owner":"jerry"},"version":"21:0"}

查询历史

        peer chaincode query -C mychannel -n marbles02 -c '{"Args":["getHistoryForMarble","marble01"]}'
        返回结果
        "Value":{"docType":"marble","name":"marble01","color":"red","size":1000,"owner":"tom"}, "Timestamp":"2017-07-19 08:56:22.754146148 +0000 UTC", "IsDelete":"false"},{"TxId":"b15e0afd4df8d8553ea35c68a69382091cd3fe319b828fa6199cd7022cee601d", "Value":{"docType":"marble","name":"marble01","color":"green","size":9999,"owner":"jerry"}, "Timestamp":"2017-07-19 09:07:15.293083849 +0000 UTC", "IsDelete":"false"}]

##### 个人总结如下：
对CouchDB数据库的更改都是有效的，并且链数据和库数据是同步完成修改的。但是区块连历史中却没有这个修改的记录。修改的数值再下一次交易会生效，而整个历史过程中，也没有找到该部分修改记录。


### 其它命令

    peer chaincode invoke -o orderer0:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marbles02 -c '{"Args":["initMarble","marble1","blue","35","tom"]}'

    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marbles02 -c '{"Args":["transferMarble","marble2","jerry"]}'

    peer chaincode invoke -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marbles02 -c'{"Args":["transferMarblesBasedOnColor","blue","jerry"]}'

    peer chaincode invoke -o -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n marbles02 -c '{"Args":["delete","marble1"]}'

    peer chaincode query -C mychannel -n marbles02 -c '{"Args":["readMarble","marble2"]}'

    root@35aa4ce0eb9f:/opt/gopath/src/github.com/hyperledger/fabric/peer# peer chaincode query -C mychannel -n marbles02 -c '{"Args":["getHistoryForMarble","marble2"]}'

    peer chaincode query -C mychannel -n marbles02 -c '{"Args":["queryMarblesByOwner","jerry"]}'
