# Example2 Cli Test

#### Query A & Query B (Cli)
    peer chaincode query -C channeltest -n mycc -c '{"Args":["query","b"]}'
    peer chaincode query -C channeltest -n mycc -c '{"Args":["query","a"]}'
#### A invoke B
    peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C  mychannel -n mycc -c '{"Args":["invoke","a","b","150"]}'
#### Install Cahincode
    peer chaincode install -n devincc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
#### instantiate Cahincode
    /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C  mychannel -n mycc1  -v 1.0  -c '{"Args":["init","a","100","b","200"]}' -P "OR      ('Org1MSP.member','Org2MSP.member')"
