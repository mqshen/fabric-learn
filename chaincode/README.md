chaincode 安装，实例化及更新等操作，最终调用的是系统 chaincode lccc（core/scc/lccc/lccc.go）

### install    
对应的是 lccc 中的 executeInstall
包含好部署信息的 deploy special（ChaincodeDeploymentSpec）是作为第二个参数传入的。     
然后按照     

    path := fmt.Sprintf("%s/%s.%s", chaincodeInstallPath, ccname, ccversion)    

这样的规则获得的路径来保存 这个传入的 chaincode package 。

### instantiate   
传入的参数
1. deploy 字符串
2. chain name
3. ChaincodeDeploymentSpec
4. policy 如果没有则设为该 chain 的默认背书策略     
5. escc
6. vscc

对应的是 lccc 中的 executeDeploy     
权限判断    

    lccc.acl(stub, chainname, cds)  //目前没有实现

读取之前存储的 chaincode 然后调用    

    lccc.createChaincode(stub, chainname, cds.ChaincodeSpec.ChaincodeId.Name, cds.ChaincodeSpec.ChaincodeId.Version, depSpec, policy, escc, vscc)

来创建。 这个 createChaincode 函数主要功能是吧 chaincode package put 到 stub 中。

执行完后，在 endsorer.go 的 callChaincode 中判断当前是不是部署交易。如果是则调用

    chaincode.Execute(ctxt, cccid, cds)

来启动对应的 chaincode 。

处理容器的分为以下几种：    
1. CreateImageReq 部署chaincode     
2. StartImageReq  启动chaincode     
3. StopImageReq   停止chaincode     
4. DestroyImageReq 销毁chaincode    
