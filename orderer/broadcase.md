### Handle    
1. 验证接受到的信息的channel header     
2. 如果消息为HeaderType_CONFIG_UPDATE类型的，调用supportManager的Process函数来得到处理修饰过的消息。    
3. 根据交易头中的channel id从support manager中获取对应的support。    
4. 验证是否为正常的交易 _, filterErr := support.Filters().Apply(msg)     
5. 把接受到的消息推入队列 support.Enqueue(msg)      

### supportManager的Process     
由于broadcase handler的support manager由     

    broadcastSupport{
	    Manager:               ml,
		ConfigUpdateProcessor: configupdate.New(ml.SystemChannelID(), configUpdateSupport{Manager: ml}, signer),
	}

创建，所以Process实际调用的是orderer/configupdate/configupdate.go 中的Processor类型的Process函数。     
从processor中的manager获取chain并调用newChannelConfig来更新配置信息。    

### 消息入队    
chainsupport包含的chain是由    

    cs.chain, err = consenter.HandleChain(cs, metadata)    

来创建的。support.Enqueue(msg) 

    func (cs *chainSupport) Enqueue(env *cb.Envelope) bool {
	    return cs.chain.Enqueue(env)
    }

实际调用的是chainsupport包含的chain的Enqueue方法。    


### 队列消息处理    


