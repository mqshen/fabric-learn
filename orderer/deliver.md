#### Handle

peer 的 gossip 服务会向对应的 orderer 注册 block 监听服务。

它会循环从 <-cursor.ReadyChan() 读取入链的 block 并通过 sendBlockReply(srv, block) 向注册上来的 deliver client 发送 block 。
