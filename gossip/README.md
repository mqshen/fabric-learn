gossip service 是在 chain 创建的时候启动。（core/peer/peer.go）     

    service.GetGossipService().InitializeChannel(cs.ChainID(), c, ordererAddresses)

InitializeChannel 启动 state provider

    g.chains[chainID] = state.NewGossipStateProvider(chainID, g, committer, g.mcs)

为 chain 创建 state provider 。
向对应的 channel 投递 order 信息:     

    // Deliver in order messages into the incoming channel
    go s.deliverPayloads()

接收 payload 并由 committer 向对应账本提交 block。

    d.clients[chainID] = blocksprovider.NewBlocksProvider(chainID, abc, d.gossip, d.mcs)

从 orderer 服务获取 block 由 blocksprovider 的 DeliverBlocks 来把它们传入 gossip service 的待处理 payload 队列中。
