#### ProcessProposal     
core/endorser/endorser.go->ProcessProposal     
core/peer/msgvalidation.go->ValidateProposalMessage  //验证proposal并返回proposal      
core/endorser/endorser.go->checkACL                  //权限校验，目前处在FAB-2457- we need to fix this right
core/endorser/endorser.go->getTxSimulator            //根据proposal提供的ChainID获取transaction simulator     
core/endorser/endorser.go->simulateProposal          //simulate chaincode调用      
core/endorser/endorser.go->endorseProposal           //通过调用ESCC来为proposal背书      
解析调用endorseProposal返回的信息，并把包含在pResp.Response.Payload中的chaincode调用返回给客户端


#### getTxSimulator     
core/ledger/kvledger/kv_ledgers.go->GetLedger        //根据chainName获取ledger
core/ledger/kvledger/kv_ledger.go->NewTxSimulator()  //获取transaction simulator

###### GetLedger      
从lManager中获取对应chainName的ledger。这些ledgers是通过peer node实例化的使用通过调用:    
core/ledger/kvledger/kv_ledgers.go->CreateLedger     
来创建的。


###### NewTxSimulator    
调用ledger对应的tx manager来创建transaction simulator

#### simulateProposal    
1. 获取proposal的ChaincodeInvocationSpec     
[comment]: //2. 检查ACL    
3. 检查chaincode的ESCC和VSCC    
4. 执行proposal（callChaincode）    
5. 获取模拟执行结果    

###### callChaincode(判断是部署或是调用交易)    
1. 判断是否为系统chaincode     
2. 创建chaincode context     
3. 执行chaincode（chaincode.ExecuteChaincode）     
4. 如果是部署或更新那就根据ChaincodeSpec中指定的Name创建新的chaincode context，并部署（e.deploy）     

#### e.deploy     
1. 调用chaincodeSupport部署chaincode(chaincodeSupport.Deploy)    
2. 调用chaincodeSupport启动chaincode(chaincodeSupport.Launch)    
3. 调用chaincodeSupport停止chaincode(chaincodeSupport.Stop)    

#### chaincodeSupport.Deploy    
1. 获取runningChaincodes锁      
2. 如果chaincode在运行就停止部署，并释放锁    
3. 释放runningChaincodes锁      
4. 获取调用的环境及参数    
5. 获取请求代码包，并创建container.CreateImageReq    
6. 获取虚拟机类型，并调用container.VMCProcess创建image及container     

#### container.VMCProcess    
1. 根据类型创建对应vmcontroller    
2. 创建goroutine，调用req.do  (CreateImageReq.do) 部署交易    

#### dockercontroller.Deploy      
1. 创建docker client    
2. 调用vm.deployImage部署image    

#### chaincodeSupport.Launch    
1. 判断chaincode是否已经启动     
2. 如果是系统container或不是dev模式，就启动container（chaincodeSupport.launchAndWaitForRegister）     
3. 初始化并等到chaincode准备好（chaincodeSupport.sendInitOrReady）     

#### chaincodeSupport.Stop     
1. 构造container.StopImageReq     
3. 调用container.VMCProcess停止container      
2. 判断chaincode是否已经启动     
3. 从runningChaincodes.chaincodeMap中删除对应的chaincode name     

#### chaincodeSupport.launchAndWaitForRegister     
1. chaincode启动前的准备（chaincodeSupport.preLaunchSetup）     
2. 获取chaincode的环境及参数     
3. 构造container.StartImageReq      
4. 调用container.VMCProcess启动container      

#### chaincodeSupport.preLaunchSetup
1. 创建chaincode runtime enviroment，并加入runningChaincodes.chaincodeMap     

#### StartImageReq.do     
1. 调用对应的vm的Start方法     

#### dockercontroller.Start     
1. 创建新的docker client    
2. 调用vm.stopInternal停止
3. 调用vm.createContainer创建container     
4. 调用docker client的StartContainer启动container     

#### vm.createContainer     
下面只说下dockercontroller的createContainer方法：    
1. 构造docker.Config和docker.CreateContainerOptions    
2. 调用docker client的CreateContainer创建container     
