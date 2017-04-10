### 启动     
peer通过使用命令：      

    build/bin/peer node start

启动。调用的是peer/node/start.go->serve(args []string)

启动步骤:
1. 根据系统所配置的监听地址，建立TCP监听。
2. 创建event hub server。
3. 创建grpc server。
4. 向grpc server注册chaincode support server
5. 向grpc server注册admin server
6. 创建endorser server并注册到grpc server
7. 创建Chain（createChain(chainID)）
8. 初始化gossip服务     
9. 创建deliver服务     
10. 创建ledger commiter
11. 创建或获取创世区块     
12. commiter和区块加入deliver渠道     
13. 启动grpc server
14. 启动event hub server

##### 注册chaincode support server     
1. 创建Chaincode Support实例(chaincode.NewChaincodeSupport)     
2. 注册系统chaincode     
3. 向grcp server注册chaincode support实例     

##### 创建Chain      
创建chainID对应的账本：    
1. 新建legder配置信息     
2. 创建legder

###### 创建legder     
1. 创建文件系统的block store    
2. 根据配置创建对应的transaction manager     
3. 通过重新提交最后的有效区块来恢复状态数据库    


