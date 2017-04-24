launchAndWaitForRegister 是启动 chaincode 的入口。

#### chaincodeSupport.launchAndWaitForRegister
1. chaincode启动前的准备（chaincodeSupport.preLaunchSetup）     
2. 获取chaincode的环境及参数     
3. 构造container.StartImageReq      
4. 调用container.VMCProcess启动container  

#### chaincodeSupport.preLaunchSetup
1. 创建chaincode runtime enviroment，并以 chaincode 为 key 加入runningChaincodes.chaincodeMap  
2. 返回等待的 notfy chan。

#### VMProcess （docker为例）
构造 dockercontroller.DockerVM ， 然后调用 StartImageReq.do 。     
第一次启动启动时，由于还没有该 image id， 需要新创建。    

    vm.deployImage(client, ccid, args, env, reader)

来部署 chaincode ， 之后使用     

    vm.createContainer(ctxt, client, imageID, containerID, args, env, attachStdout)    

来创建 chaincode ， 并用    

    client.StartContainer(containerID, nil)

来启动 chaincode 。这时 docker 中的 chaincode 启动

### Start (core/chaincode/shim/chaincode.go)    
这个函数调用

    stream, err := chaincodeSupportClient.Register(context.Background())

来向 peer 注册本 chaincode （peer 会调用 Register ） 同时调用

    err = chatWithPeer(chaincodename, stream, cc)

来进行通讯，这会调用    

    handler.serialSend(&pb.ChaincodeMessage{Type: pb.ChaincodeMessage_REGISTER, Payload: payload})

来通知 peer chaincode 启动完成。

### Register （core/chaincode/chaincode_support.go）    

    chaincodeSupport.HandleChaincodeStream -> HandleChaincodeStream     

调用    

    handler := newChaincodeSupportHandler(chaincodeSupport, stream)

来创建新的对应此 chaincode 的 handler 。
