### 先决条件    
1. 部署 chaincode 需要 admin 权限，需要把部署用户的证书写入到 msp config 的admincerts 目录下    
2. 验证机构的根证书，把根证书放入 cacerts 目录后，还需要在 config.yaml 中加入对应的配置如：    

    OrganizationalUnitIdentifiers:
      - Certificate: "cacerts/cacert.pem"
        OrganizationalUnitIdentifier: "COP"
      - Certificate: "cacerts/ca-cert.pem"
        OrganizationalUnitIdentifier: "fabric-server"

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
7. 初始化gossip服务     
8. 如果peer-defaultchain没有设置为false则会创建genesis block，然后由这个创世区块来创建Chain（CreateChainFromBlock(block)）
9. peer初始化
10. 启动grpc server
11. 启动event hub server

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
