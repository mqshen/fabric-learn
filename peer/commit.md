当 peer 的 gossip 接收到 Payloads 时，会执行 kvledger 的 Commit 此方法会调用

    err = l.txtmgmt.ValidateAndPrepare(block, true)

来验证当前区块数据的合法性如： MVCC 校验，这会调用 validator/statebasedval/state_based_validator.go 的 ValidateAndPrepareBatch
