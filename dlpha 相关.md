
## DLPHA问题

##### 问题1： gas代付
1inch gas refund

https://www.chainnews.com/news/693728375768.htm

https://blog.1inch.io/the-1inch-foundation-will-distribute-10-mln-1inch-to-the-defi-community-as-gas-cost-refunds-2dd35a6a084f

https://www.chainnews.com/news/737522892553.htm

##### 问题2:  引入新的Defi项目
每个具体的项目需要实现自己的Adapter合约，该合约继承自AdapterBase2，每个项目都要开发一遍

##### 问题3:  资产的白黑名单
原来的Policy中有AssetBlackList 和 AssetWhiteList


## SakePerp集成Enzyme系统

### 示意图: https://drive.google.com/file/d/1lWgfyhlhVpRAMuafRHYHteFv6S2TA9oG/view?usp=sharing

### 步骤说明:

1. 每一个地址通过该系统去Sakeperp里开仓都会创建一个新的开仓合约，通过该开仓合约对SakePerp进行OpenPosition, ClosePosition, AddMargin, RemoveMargin等操作。

2. 开仓时busd会先转进这个开仓合约然后再去Sakeperp开仓，开仓后剩余的钱会转回给操作地址。

3. 开仓合约继承自ERC20，大小跟仓位有关系，在开仓完成后，会mint出对应大小的代币转移给开仓的地址。

4. 开仓合约拥有一个owner，owner有add margin, remove margin的权限，开仓合约记录自己的仓位信息(交易对，大小，方向等)。

5. 当投资者在Enzyme系统退出的时候，会将等比例的开仓合约代币转给它，他自行通过开仓系统进行平仓获取收益。

6. 如果开仓合约的仓位爆掉，则开仓合约的价值为0，如果用户退出enzyme，基金经理还往开仓合约里增加了保证金，则用户会多获取收益？？

7. 目前合约中没有原生支持的部分平仓的接口，这部分考虑是否可以移植PerpV2。