### Create    
    cf, err = InitCmdFactory(EndorserNotRequired, OrdererRequired)

初始化cmd factory ， 执行 sendCreateChainTransaction 来根据制定的 channelTxFile 或默认设置来创建 channel 的配置信息。
检查这个配置信息，并向 orderer 广播这个信息。

### Command    

channel的join命令会读取由create命令所创建的block，然后按调用chaincode的规则，设chain id为空，来调用cscc这个系统chaincode。

### Server    
