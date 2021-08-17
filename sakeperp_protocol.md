## SakePerp Protocol

github地址: https://github.com/Sakeswap/sakeperp-protocol

### 概述
主要完成了Sakeperp, SystemSettings, SakePerpState, SakePerpViewer 合约的开发

### 主要合约

### SakePerp
地址: https://github.com/qlcchain/sakeperp/blob/main/contracts/SakePerp.sol

sakePerp合约是Trader交易的入口，它的功能主要分为三大类，面向trader的交易接口，面向keeper的收费接口，以及查询接口

#### trader交易接口

| 方法                 | 作用                        |
| ------------------- | ---------------------------  | 
| OpenPosition        | 用户开仓                     |
| ClosePosition       | 用户平仓                |
| AddMargin           | 增加保证金                |
| RemoveMargin        | 移除保证金                    |
| settlePosition      | 关闭状态下平仓                |

#### Keeper收费接口
| 方法                 | 作用                        |
| ------------------- | --------------------------- |
| liquidate        | 清算用户                       |
| payFunding       | 支付funding fee                |
| payOvernightFee  | 支付overnight fee              |

#### 查询接口
| 方法                 | 作用                        |
| ------------------- | --------------------------- |
| getMarginRatio        | 获取保证金率               |
| getPosition        | 获取仓位信息               |
| getPositionNotionalAndUnrealizedPnl        | 获取名义头寸和pnl               |
| getLatestCumulativePremiumFraction        | 获取资金费率               |
| getLatestCumulativeOvernightFeeRate        | 获取过夜费率               |

### SystemSettings
地址: https://github.com/qlcchain/sakeperp/blob/main/contracts/SystemSettings.sol

SystemSettings 主要用于各种配置项的修改和查询

| 属性                 | 作用                        |
| -------------------  | --------------------------- |
| insuranceFundFeeRatio        | 开仓/平仓手续费 MM和insurance Fund分成比例               |
| lpWithdrawFeeRatio        | LP退出扣除比例               |
| overnightFeeRatio        | overnight fee的收费比率         |
| overnightFeeLpShareRatio        | overnight fee MM和insurance Fund分成比例               |
| fundingFeeLpShareRatio        | funding fee MM和insurance Fund分成比例               |
| exchangeMap | exchange列表|


### SakePerpState
地址: https://github.com/qlcchain/sakeperp/blob/main/contracts/SakePerpState.sol

SakePerpState 主要负责完成了trader开仓平仓的时间限制

| 属性                 | 作用                        |
| -------------------  | --------------------------- |
| tradingState        | trader开仓时间记录表               |
| waitingWhitelist        | 交易白名单 不受开仓间隔限制               |
| allWaitingWhitelist        | 交易白名单 不受开仓间隔限制               |

#### 限制交易接口

| 方法                 | 作用                        |
| ------------------- | --------------------------- |
| checkWaitingPeriod        | 用户是否可交易 |
| setWaitingPeriodSecs       | 用户交易冷却等待时间                |


### SakePerpViewer
地址: https://github.com/qlcchain/sakeperp/blob/main/contracts/SakePerpViewer.sol

SakePerpViewer 主要是一些聚合性的读取接口

| 方法                 | 作用                        |
| ------------------- | --------------------------- |
| getUnrealizedPnl        | 获取未实现pnl |
| getPersonalBalanceWithFundingPayment | 获取计算了FundingFee 和 OvernightFee 的余额 |
| getPersonalPositionWithFundingPayment | 获取计算了FundingFee 和 OvernightFee 的仓位信息 |
| getMarginRatio | 获取保证金率 |
| getMarginRatios | 批量获取保证金率 |
| getFundingAndOvernightFee | 获取FundingFee 和 OvernightFee的实时数据|

### AirDrop
地址: https://github.com/qlcchain/sakeperp/blob/main/contracts/Airdrop.sol

AirDrop 主要是批量给用户转账

| 方法                 | 作用                        |
| -------------------  | --------------------------- |
| batchTransfer        | 批量转账 |
