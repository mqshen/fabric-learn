block 是由 WriteBlock（orderer/multichain/chainsupport.go） 写入区块的。  

    err := cs.ledger.Append(block)

向账本添加 block 。 ledger 是由 orderer.yaml 配置的。有 ramledger，fsledger 。

它的 deliver server 负责给 gossip 中的 deliver client 发送 block
