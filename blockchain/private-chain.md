## 区块链私链搭建
由于工作原因，需要搭建各种私链用于自动化测试。私链的好处有:
- 转账耗时短，时间可配置
- 不依赖水龙头
- 可以灵活配置创世区块

本文收集记录下各种私链的搭建过程，使用方法可参考原始文档。

### 支持的区块链(docker部署)
|区块链| 原始文档 | 命令|
|--|--|--|
| BTC | [https://github.com/hunterlong/btcregtest-insight](https://github.com/hunterlong/btcregtest-insight) |docker run -it -p 3001:3001 hunterlong/btcregtest-insight:latest
| BTC-USDT(*) | [https://github.com/ww8912188/btc-usdt-regtest](https://github.com/ww8912188/btc-usdt-regtest) | docker run -d -p 3001:3001 -p 8431:8431 -p 8432:8432 ww8912188/btc-usdt-regtest
| LTC | [https://github.com/hunterlong/ltcregtest-insight](https://github.com/hunterlong/ltcregtest-insight) | docker run -it -p 3005:3005 hunterlong/ltcregtest-insight:latest
| VET | [https://github.com/vechain/thor](https://github.com/vechain/thor)  |docker run -d -v /opt/data/vechain/org.vechain.thor:/root/.org.vechain.thor -p 8669:8669 -p 11235:11235 -p 11235:11235/udp --name thor-node vechain/thor solo --persist --api-addr 0.0.0.0:8669

(*) 以上btc私链和btc-usdt区别在于后者会在相同的一条链上同时跑btc和omni usdt，并分别提供REST和RPC服务。

### 非容器部署
ETH, COSMOS, BCH暂时还没容器化，先写下部署步骤及资料。后续有空会做成容器，方便后续使用。

#### 部署ETH private chain
使用了[源码]([https://github.com/ethereum/go-ethereum](https://github.com/ethereum/go-ethereum))编译，主要参考了这篇[文章](https://medium.com/cybermiles/running-a-quick-ethereum-private-network-for-experimentation-and-testing-6b1c23605bce)，简单罗列下步骤:

1.  install go, 编译geth
```
cd /opt/go-ethereum
make geth
```
2. add geth to ENV
```
export PATH="$PATH:/opt/go-ethereum/build/bin"
```
3. generate account
```
cd ~
mkdir gethDataDir
geth account new --datadir ~/gethDataDir
```
4. generate genesis.json
```
{
  "config": {
    "chainId": 12345,
    "homesteadBlock": 0,
    "eip155Block": 0,
    "eip158Block": 0
  },
  "difficulty": "1",
  "gasLimit": "2100000",
  "alloc": {
    "0x4fe77cb0cdc5d302778a4096fad97c5c5313d4b5": {
      "balance": "20000000000000000000"
    },
    "0xd0da98b9d11a6632e0f534a5ff32152c8bc26629": {
      "balance": "20000000000000000000"
    }
  }
}
```
5. initialize blockchain
```
cd ~/gethDataDir
geth --datadir ~/gethDataDir/ init genesis.json
```
6. start ethereum
```
cd ~
geth --mine --minerthreads=1 --datadir ~/gethDataDir --networkid 12345 --rpc --rpcaddr 0.0.0.0 --rpcport=8547
```
7.  geth交互
```
geth attach ipc:gethDataDir/geth.ipc
```
enable RPC可以参考[这篇](https://gist.github.com/fishbullet/04fcc4f7af90ee9fa6f9de0b0aa325ab)文章。
> geth invoke option:
  --rpc                  Enable the HTTP-RPC server
  --rpcaddr value        HTTP-RPC server listening interface (default: "localhost")
  --rpcport value        HTTP-RPC server listening port (default: 8545)
8. ETH转账可以参考[这里](http://blog.bradlucas.com/posts/2017-08-17-send-eth-from-geth-console/)
```
# 列举所有accounts
personal.listAccounts
# 获取balance
web3.fromWei(eth.getBalance(eth.coinbase))
# unlock
personal.unlockAccount(eth.coinbase)
# transfer
eth.sendTransaction({from:eth.coinbase, to:"0x5182fc91bd7ec4703a9244a69bbe44d9e91059f5", value: web3.toWei(1, "ether")})
```
9. 关于chain ID的说明可以参考[这里](https://ethereum.stackexchange.com/questions/17051/how-to-select-a-network-id-or-is-there-a-list-of-network-ids)

#### 部署ERC20
1. generate abi and bin using [remix](http://remix.ethereum.org/#optimize=false&version=soljson-v0.4.25+commit.59dbf8f1.js)
2. 使用geth部署
```
geth account list --datadir ./keystore/
geth account import --datadir /root/gethDataDir/keystore/ /root/gethDataDir/key.pri
geth account list

# attch
geth attach ipc:geth.ipc
personal.listAccounts  -> should contain account 0xc5E33b3CED9aF1c58875759cE1A179b9Ee06761d

personal.unlockAccount("0xc5E33b3CED9aF1c58875759cE1A179b9Ee06761d")
# deploy
loadScript('/opt/nash/Simple.abi')
loadScript('/opt/nash/Simple.bin')
```
3. 附上[如何导入private key](https://github.com/ethereum/go-ethereum/wiki/Managing-your-accounts)文档

#### 部署COSMOS private chain
cosmos单节点私链不太好模拟真实的stake功能，写了一个shell，先后起两个validator, 后一个node join前面node的网络。具体可以参考cosmos文件夹。
