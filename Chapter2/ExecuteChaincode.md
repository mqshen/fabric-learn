core/chaincode/chaincodeexec.go->ExecuteChaincode

创建chaincode invocation spec（CIS）      

core/chaincode/exectransaction.go->Execute     

如果不是部署chaincode那就是执行chaincode invoke    

theChaincodeSupport.Launch 启动chaincode     
createTransactionMessage创建transaction message      
theChaincodeSupport.Execute 执行chaincode      

#### chaincode_support.go->Execute      
1. 根据chaincode id获取chaincode runtime enviroment->chrte      
2. 调用chrte.handler.sendExecuteMessage向chaincode handler发送执行消息    


#### handler.sendExecuteMessage(core/chaincode/handler.go)     
1. 调用handler.createTxContext创建transaction context     
2. 调用handler.setChaincodeProposal //TODO状态     
3. 如果是transaxtion的话调用handler.triggerNextState触发FSM的下一状态     

#### handler.triggerNextState    
它会向handler.nextState这个channel发送&nextStateInfo{msg, send}，这会触发handler的processStream方法。然后触发handler.HandleMessage方法来处理message。    
HandleMessage向handler的FSM传递接受到的msg事件，对于chaincode的invoke对应的是执行enterBusyState方法    

#### handler.enterBusyState    
下面只解释ChaincodeMessage_INVOKE_CHAINCODE分支：    
创建chaincode context
调用createTransactionMessage创建chaincode message    
调用handler.chaincodeSupport.Execute执行chaincode    

core/chaincode/shim/handler.go handleInit
handleInvokeChaincode调用chaincode

handleTransaction是真正的调用chaincode

err = creator.Verify(msg, sig)
2016-12-20 19:33:12.462 CST [chaincode] processStream -> ERRO 0c9 Got error: transaction not found bonust/**TEST_CHAINID**

core/chaincode/shim/chaincode.go Start 注册与peer与chaincode通讯
