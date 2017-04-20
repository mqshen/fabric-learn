core/chaincode/chaincodeexec.go->ExecuteChaincode

创建chaincode invocation spec（CIS）      

#### core/chaincode/exectransaction.go->Execute     

如果不是部署chaincode就把消息类型设置为 ChaincodeMessage_TRANSACTION

theChaincodeSupport.Launch 获取 chaincode runtime enviroment    
createCCMessage 创建 chaincode message      
theChaincodeSupport.Execute 执行chaincode      

#### chaincode_support.go->Launch    
判断 chaincode 是否已经启动，如果未启动则尝试启动
根据 chaincodeName 从的 runningChainCodes 中获取 chaincode runtime enviroment(chrte)      
如果此 chaincode 还没有启动，且不是系统 chaincode 那么读取部署时的交易，从中获取 deploy payload 并重新启动注册

#### chaincode_support.go->Execute      
1. 根据chaincode id获取chaincode runtime enviroment->chrte      

    chaincodeSupport.chaincodeHasBeenLaunched(canName)

2. 调用chrte.handler.sendExecuteMessage向chaincode handler发送执行消息    


#### handler.sendExecuteMessage(core/chaincode/handler.go)     
1. 调用handler.createTxContext创建transaction context     
2. 调用handler.setChaincodeProposal //TODO状态     
3. 如果是transaxtion的话调用handler.triggerNextState触发FSM的下一状态     

#### handler.triggerNextState    
它会向handler.nextState这个channel发送&nextStateInfo{msg, send}，这会触发handler的HandleMessage方法来处理ccMsg。    
HandleMessage 向 handler 的 FSM 传递接受到的 msg 事件。这会针对不同的当前状态及事件进行对应的回调。    

    fsm.Callbacks{
			"before_" + pb.ChaincodeMessage_REGISTER.String():           func(e *fsm.Event) { v.beforeRegisterEvent(e, v.FSM.Current()) },
			"before_" + pb.ChaincodeMessage_COMPLETED.String():          func(e *fsm.Event) { v.beforeCompletedEvent(e, v.FSM.Current()) },
			"after_" + pb.ChaincodeMessage_GET_STATE.String():           func(e *fsm.Event) { v.afterGetState(e, v.FSM.Current()) },
			"after_" + pb.ChaincodeMessage_GET_STATE_BY_RANGE.String():  func(e *fsm.Event) { v.afterGetStateByRange(e, v.FSM.Current()) },
			"after_" + pb.ChaincodeMessage_GET_QUERY_RESULT.String():    func(e *fsm.Event) { v.afterGetQueryResult(e, v.FSM.Current()) },
			"after_" + pb.ChaincodeMessage_GET_HISTORY_FOR_KEY.String(): func(e *fsm.Event) { v.afterGetHistoryForKey(e, v.FSM.Current()) },
			"after_" + pb.ChaincodeMessage_QUERY_STATE_NEXT.String():    func(e *fsm.Event) { v.afterQueryStateNext(e, v.FSM.Current()) },
			"after_" + pb.ChaincodeMessage_QUERY_STATE_CLOSE.String():   func(e *fsm.Event) { v.afterQueryStateClose(e, v.FSM.Current()) },
			"after_" + pb.ChaincodeMessage_PUT_STATE.String():           func(e *fsm.Event) { v.enterBusyState(e, v.FSM.Current()) },
			"after_" + pb.ChaincodeMessage_DEL_STATE.String():           func(e *fsm.Event) { v.enterBusyState(e, v.FSM.Current()) },
			"after_" + pb.ChaincodeMessage_INVOKE_CHAINCODE.String():    func(e *fsm.Event) { v.enterBusyState(e, v.FSM.Current()) },
			"enter_" + establishedstate:                                 func(e *fsm.Event) { v.enterEstablishedState(e, v.FSM.Current()) },
			"enter_" + readystate:                                       func(e *fsm.Event) { v.enterReadyState(e, v.FSM.Current()) },
			"enter_" + endstate:                                         func(e *fsm.Event) { v.enterEndState(e, v.FSM.Current()) },
		}    

对应 chaincode 的 deploy 是执行 enterReadyState 方法
对应 chaincode 的 invoke 是执行 enterBusyState 方法    

#### handler.enterBusyState    
下面只解释ChaincodeMessage_INVOKE_CHAINCODE分支：    
创建chaincode context
调用createTransactionMessage创建chaincode message    
调用handler.chaincodeSupport.Execute执行chaincode    

core/chaincode/shim/handler.go handleInit
handleInvokeChaincode调用chaincode

handleTransaction是真正的调用chaincode

#### serialSend      
core/chaincode/shim/handler.go

    //serialSend serializes msgs so gRPC will be happy
    func (handler *Handler) serialSend(msg *pb.ChaincodeMessage) error {
        handler.serialLock.Lock()
        defer handler.serialLock.Unlock()

        err := handler.ChatStream.Send(msg)

        return err
    }

这个函数把 chaincode 消息发送到 chaincode 由 chaincode 的 handler 来处理。这里的 ChatStream 是由在 chaincode 启动时创建的
系统 chaincode：     

    stream := newInProcStream(recv, send)

非系统 chaincode：

    clientConn, err := newPeerClientConnection()
	  if err != nil {
	  	chaincodeLogger.Errorf("Error trying to connect to local peer: %s", err)
	  	return fmt.Errorf("Error trying to connect to local peer: %s", err)
	  }

	  chaincodeLogger.Debugf("os.Args returns: %s", os.Args)

	  chaincodeSupportClient := pb.NewChaincodeSupportClient(clientConn)

	  // Establish stream with validating peer
	  stream, err := chaincodeSupportClient.Register(context.Background())

core/chaincode/shim/chaincode.go Start 注册与peer与chaincode通讯

通过 handler.serialSendAsync(in, errc) 把消息发给 chaincode 。

err = creator.Verify(msg, sig)
2016-12-20 19:33:12.462 CST [chaincode] processStream -> ERRO 0c9 Got error: transaction not found bonust/**TEST_CHAINID**

chaincode 启动是由 docker 中 main 函数触发的。
