### PutState(key, value)    
把 value 写入账本 key 中     

    respChan, uniqueReqErr := handler.createChannel(txid)

根据 txid 创建通信 channel ，并向 peer 发送 put_state 的消息

    esponseMsg, err := handler.sendReceive(msg, respChan)

    handler.serialSendAsync(msg, errc)

当 peer 的 handler 收到此消息时调用     

    func (handler *Handler) HandleMessage(msg *pb.ChaincodeMessage) error {

来处理消息，实际调用的是 enterBusyState 它首先会判断是否还是有统一个交易号的东西在处理，在检查是否是有效的交易，并获取交易上下文：

    txContext, triggerNextStateMsg = handler.isValidTxSim(msg.Txid, "[%s]No ledger context for %s. Sending %s",
      shorttxid(msg.Txid), msg.Type.String(), pb.ChaincodeMessage_ERROR)

然后调用

    putStateInfo := &pb.PutStateInfo{}
    unmarshalErr := proto.Unmarshal(msg.Payload, putStateInfo)
    if unmarshalErr != nil {
      payload := []byte(unmarshalErr.Error())
      chaincodeLogger.Debugf("[%s]Unable to decipher payload. Sending %s", shorttxid(msg.Txid), pb.ChaincodeMessage_ERROR)
      triggerNextStateMsg = &pb.ChaincodeMessage{Type: pb.ChaincodeMessage_ERROR, Payload: payload, Txid: msg.Txid}
      return
    }

    err = txContext.txsimulator.SetState(chaincodeID, putStateInfo.Key, putStateInfo.Value)

来设置状态。这会调用 lockBasedTxSimulator 的 SetState

    // SetState implements method in interface `ledger.TxSimulator`
    func (s *lockBasedTxSimulator) SetState(ns string, key string, value []byte) error {
        s.helper.checkDone()
        s.rwsetBuilder.AddToWriteSet(ns, key, value)
        return nil
    }

这会获取对应 namespace 的 read write set ，并向其中的 writeMap 添加新的 key->value 。

DelState 方法的过程类似。
