### 启动     
orderer通过使用命令：      

    build/bin/orderer

启动。调用的是orderer/main.go->main()

启动步骤:
1. 根据系统所配置的监听地址，建立TCP监听。
2. 根据之前建立的TCP监听地址，创建GRPC服务。    
3. 根据配置信息加载msp信息。     
4. 创建账本工厂（ledger factory）。    
5. 如果还没有链数据，则创建创世区块。    
6. 创建各共识插件实例。    
7. 创建signer与多链管理的实例。    
8. 新建server并注册到grpc server上。     

#### 账本工厂（ledger factory）     
账本有文件（fileledger），json（jsonledger），内存（ramledger）这几种类型。

#### 多链管理    
##### 初始化
根据chain id获取或创建readwriter    

    ledgerFactory.GetOrCreate(chainID)    

获取配置信息     
    
    configTx := getConfigTx(rl)

> 获取账本的最后一个区块     
> 从这个区块获取最后一次修改配置的区块号
> 获取

根据配置信息创建ledgerResources    

    ledgerResources := ml.newLedgerResources(configTx)

> 创建config manager
    
    configtx.NewManagerImpl(configTx, initializer, nil)

> 如果设置了ChainCreationPolicyNamesKey则为系统链，则调用createSystemChainFilters来创建chain support。其他则使用createStandardFilters来创建


新建的server中包含     
* deliver handler: 接口封装好的包含SeekInfo信息的DELIVER_SEEK_INFO类型消息，并返回对应的block信息。     
* broadcast handler: 按次序接收已经确认好的common.Envolope，返回成功或提示失败。     


