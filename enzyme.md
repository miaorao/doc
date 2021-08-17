## EnzymeFinance
github地址: https://github.com/enzymefinance/protocol

### 概述
Enzyme 是一个部署在ETH上的链上资金管理平台，由基金管理人员创建Fund，投资人投入资金，其后基金操作员将资金买入链上各种资产，依靠各类资产的升值为投资人获取收益。

### 结构图

https://drive.google.com/file/d/1g0Q37q586GLBPPesyOcX0-KPskveF8g8/view?usp=sharing

### 主要合约

### VaultLib

Vault 操作的逻辑功能合约，存储资金，记录 shares，继承自 ERC20，拥有 mint 和 burn 接口，屏蔽了 transfer，approve，transferfrom 接口，不能当成一般性的 ERC20 使用，

| state         | 作用                                                   |
| ------------- | ------------------------------------------------------ |
| accessor      | Controller 合约地址，VaultLib 中的功能 Controller 调用 |
| creator       | newfund 的发起者                                       |
| migrator      | 迁移合约地址                                           |
| owner         | 设置的 owner 的地址                                    |
| trackedAssets | 支持的资产的地址列表                                   |

### ComptrollerLib

Vault 买入卖出（buyShares, redeemShares）, 基金操作(callOnExtension)

| state               | 作用                        |
| ------------------- | --------------------------- |
| vaultProxy          | 所操作的 vault proxy 的地址 |
| FUND_DEPLOYER       | Fund Creator                |
| FEE_MANAGER         | Fee 操作相关                |
| INTEGRATION_MANAGER | 外部操作                    |
| POLICY_MANAGER      | 限制性检查                  |

### FeeManager

**fee 的类型**

#### EntranceRateBurnFee

在用户 BuyShares 操作完成后，将用户的 Shares burn 掉一个比例，用作入场费，给所有人增加收益

#### EntranceRateDirectFee

在用户 BuyShares 操作完成后，将用户的 Shares 抽出一个比例转移给 Fund 的 Owner

#### ManagementFee

在用户 BuyShares 和 RedeemShares 之前，Mint 一个制定比例给 Fund 的 Owner

#### PerformanceFee

在用户 BuyShares 之前，BuyShares 之后，和 redeem shares 之前，会调用 calcPerformance 计算本周期的收益，然后根据情况按比例给 VaultProxy mint shares，或者从 VaultProxy 中 burn shares，通过调用 FeeManager 中的 payout 方法可以将奖励结给 Fund 的 Owner

每个 Fee 的类型是一个单独的合约，在内部通过 **mapping(address => FeeInfo) private comptrollerProxyToFeeInfo ** 给每一个 Vault Controller 单独设置参数。

### FeeManager 功能

#### registerFees 和 deregisterFees

注册和反注册收费合约到 FeeManager 中去

#### invokeHook

在 ComptrollerLib 的 BuyShares 和 RedeemShares 的方法中会调用，执行对应的 Fee 的相关操作

#### setConfigForFund

ComptrollerLib 调用 FeeManager 的这个接口来给自己设置收费参数

#### receiveCallFromComptroller

1. Settle and update all continuous fees
2. Payout

### PolicyManager

**Policy 的类型**

#### buy-shares 相关

| 类                       | 作用                 |
| ------------------------ | -------------------- |
| BuySharesCallerWhitelist | 白名单可以 BuyShares |
| InvestorWhitelist        | 白名单可操作         |
| MinMaxInvestment         | 最大最小投资额       |

#### call-on-integration 相关

| 类                   | 作用                       |
| -------------------- | -------------------------- |
| AdapterBlacklist     | 不能用 list 中的 adapter   |
| AdapterWhitelist     | 只能使用 list 中的 adapter |
| AssetBlacklist       | 不能持有 list 中的 asset   |
| AssetWhitelist       | 只能只有 list 中的 asset   |
| GuaranteedRedemption | 带赎回期                   |
| MaxConcentration     | 控制每种资产的持有占比     |

每个 Policy 的类型是一个单独的合约，在内部为每一个 Vault Controller 单独设置参数

### PolicyManager 功能

#### registerPolicies 和 deregisterPolicies

注册和反注册条款合约到 FeeManager 中去

#### validatePolicies

在 ComptrollerLib 的 BuyShares 方法中会调用，执行对应的 Policy 的相关操作
在 IntegrationManager 的外部 defi 调用的开始 结束，会执行对应的 Policy 的相关操作 （**preCoIHook， **postCoIHook）

#### setConfigForFund

ComptrollerLib 调用 PolicyManager 的这个接口来给自己设置 Policy 参数

### FundDeployer

createNewFund 入口

### IntegrationManager

集成外部的 Defi 操作，通过 ComptrollerLib 的 callOnExtension 方法调入。

#### AdapterBase 和 AdapterBase2

封装了在 Vault Proxy 和各种 Defi Adapter 之前转移 Asset 的操作

#### Defi Adapter

每个项目的 Adapter 合约继承自 AdapterBase2，这个合约是一个纯逻辑合约，与外部的 Defi 项目进行交互，不保存有状态变量，只有一些常量，每个 Adapter 根据需要实现自己的方法，AaveAdapter 可能实现 lend 和 redeem 的功能，UniswapAdapter 可能实现 addliquidity 和 removeliquidity 以及 swap 的功能。

#### registerAdapters 和 deregisterAdapters

每个 Adapter 都是单独合约独立部署，通过注册到 IntegrationManager 的 registeredAdapters 集合中提供功能

#### addAuthUserForFund 和 removeAuthUserForFund

为某个 vault proxy 添加操作员的角色

## 主要流程

### createNewFund

合约: FundDeployer
参数:

|参数| 含义 |
|-|-|
|_fundOwner | fund 所有者 |
|_fundName| fund 名字|
|_feeManagerConfigData| fee 配置数据 |
|_policyManagerConfigData | Policy 配置数据 |


- 创建 ComptrollerProxy
- 设置 fee 配置和 policy 配置
- 创建 Vault Proxy
- 在 FeeManager PolicyManager IntegrationManager 中进行注册

### buyShares

合约: ComptrollerLib

参数:

|参数| 含义 |
|-|-|
|_buyers| The accounts for which to buy shares |
|_investmentAmounts| The amounts of the fund's denomination asset|
|_minSharesQuantities| The minimum quantities of shares to buy |

步骤:
- calcGav 计算总资产值
- PolicyManager validatePolicies BuySharesSetup 阶段检查
- FeeManager invokeHook BuySharesSetup 阶段收费
- calcGrossShareValue 计算单价
- FeeManager invokeHook PreBuyShares 阶段收费
- PolicyManager validatePolicies PreBuyShares 阶段检查
- mintShares
- transfer buyer asset to vault proxy
- FeeManager invokeHook PostBuyShares 阶段收费
- PolicyManager validatePolicies PostBuyShares 阶段检查
- calcGav
- PolicyManager validatePolicies BuySharesCompleted 阶段检查
- FeeManager invokeHook BuySharesCompleted 阶段收费

### redeemShares 和 redeemSharesDetailed

合约: ComptrollerLib

参数:

|参数| 含义 |
|-|-|
|\_sharesQuantity | The quantity of shares to redeem |
|\_additionalAssets | The amounts of the fund's denomination asset|
|\_assetsToSkip| Tracked assets to forfeit |

步骤:
- FeeManager invokeHook PreRedeemShares 阶段收费
- \_\_parseRedemptionPayoutAssets 计算给用户的资产列表
- burnShares
- Transfer payout asset to redeemer

### callOnExtension

合约: ComptrollerLib

参数:

|参数| 含义 |
|-|-|
| _extension | The Extension contract to call （FeeManager， IntegrationManager）|
| _actionId | An ID representing the action to take on the extension |
| _callArgs | The encoded data for the call |

步骤:
- 根据 contract address 转到对应合约的 receiveCallFromComptroller 方法里处理
- 这里以 IntegrationManager 为例
- isAuthUserForFund 检查交易发起者权限

| actionId | 操作                               |
| -------- | ---------------------------------- |
| 0        | \_\_callOnIntegration              |
| 1        | \_\_addZeroBalanceTrackedAssets    |
| 2        | \_\_removeZeroBalanceTrackedAssets |

步骤:

- 这里以\_\_callOnIntegration 为例，这个操作主要是调用外部 Defi 合约
- \_\_decodeCallOnIntegrationArgs 解析\_callArgs 得到要调用的 Adapeter 地址，要调用的 Adapter 的方法，以及传导的参数
- PolicyManager validatePolicies PreCallOnIntegration 阶段的检查
- 调用对应的 adapter 的对应的方法
- PolicyManager validatePolicies PostCallOnIntegration 阶段的检查