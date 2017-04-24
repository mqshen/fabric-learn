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

以sbft为例，之后会调用     

    // Enqueue accepts a message and returns true on acceptance, or false on shutdown
    func (ch *chain) Enqueue(env *cb.Envelope) bool {
    	return ch.consensusStack.backend.Enqueue(ch.chainID, env)
    }

backend的Enqueue方法,向队列添加requestEvent信息     

    // Enqueue enqueues an Envelope for a chainId for ordering, marshalling it first
    func (b *Backend) Enqueue(chainID string, env *cb.Envelope) bool {
    	requestbytes, err := proto.Marshal(env)
    	if err != nil {
    		return false
    	}
    	b.enqueueRequest(chainID, requestbytes)
    	return true
    }



### 队列消息处理(sbft为例)    
从backend获取到信息后会调用消息对象的Execute方法     

    func (r *requestEvent) Execute(backend *Backend) {
	    backend.consensus[r.chainId].Request(r.req)
    }

sbft的Request方法     

    // Request proposes a new request to the BFT network.
    func (s *SBFT) Request(req []byte) {
    	log.Debugf("replica %d: broadcasting a request", s.id)
    	s.broadcast(&Msg{&Msg_Request{&Request{req}}})
    }

实际调用Backend的Send方法    

    // Send sends to a specific SBFT peer identified by chainId and dest
    func (b *Backend) Send(chainID string, msg *s.Msg, dest uint64) {
    	if dest == b.self.id {
    		b.enqueueForReceive(chainID, msg, b.self.id)
    		return
    	}
    	b.Unicast(chainID, msg, dest)
    }

对于本机的话会直接向队列中添加

    msgEvent{chainId: chainID, msg: msg, src: src}    

msgEvent类型的Execute为    

    backend.consensus[m.chainId].Receive(m.msg, m.src)

handleRequest     

如果当前node为primary node，且node为activeView时     

    batches, committers, valid := s.sys.Validate(s.chainId, req)

> 调用backend的Validate方法，blockcutter.go->Ordered()     
>
> 获取committer，是有configManager.Validate获取committer。
>
>     committer, err := r.filters.Apply(msg)     
>
> filters是在创建mutilchain/manager.go的NewManagerImpl中的createSystemChainFilters/createStandardFilte创建的。configtxfilter.NewFilter(ledgerResources)新建的configFilter。
>
> configFilter的Apply函数验证一些消息头，并调用cf.configManager.Validate(configEnvelope)来验证消息，并返回     

    &configCommitter{
		manager:        cf.configManager,
		configEnvelope: configEnvelope,
	}

消息验证后     

    s.batches = append(s.batches, batches...)
	s.primarycommitters = append(s.primarycommitters, committers...)
	s.maybeSendNextBatch()

之后调用     

    s.sendPreprepare(batch, committers)

来进行sbft的共识阶段。    
