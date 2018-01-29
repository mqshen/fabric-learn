存储主要有 块存储（block store）， 状态存储（state store），历史存储（history store）     

### block store
fs_blockstory.go AddBlock 调用 store.fileMgr.addBlock(block) 来通过 file manager 添加 block 
