## SakePerp-Python lib

github地址: https://github.com/qlcchain/sakePerp-python

### 概述
基于Python语言的SakePerp Client Lib, 封装了SakePerp的开仓，平仓，增加保证金，移除保证金等操作

### SystemMeta
封装了从subgraph里获取数据的接口，可以根据网络类型(测试网和正式网)自动抓取正确的合约地址

| 方法                 | 作用                        |
| -------------------  | --------------------------- |
| get_contract_list    | 获取合约列表 |
| quote_asset          | quote token 地址 |
| sakeperp             | sakePerp合约地址 |
| sakeperp_vault       | sakeperp_vault 合约地址 |
| sakeperp_viewer      | sakeperpviewer 合约地址 |
| sakeperp_state       | sakeperpState 合约地址 |
| exchange             | 根据exchange的symbol获取合约地址 |
| get_exchange_list    | 获取exchange列表 |

	
### Sakeperp
封装了合约开仓，平仓，增加/减少保证金 和 一些查询功能的接口

| 方法                 | 作用                        |
| -------------------  | --------------------------- |
| open_position    | 开仓 |
| close_position   | 平仓 |
| check_wait_peroid   | 检查开仓间隔 |
| add_margin | 增加保证金 |
| remove_margin | 增加保证金 |

### Example

#### Installation

``` 
clone and install with poetry:

git clone https://github.com/qlcchain/sakePerp-python.git
cd sakePerp-python
poetry install

```

####  Initializing the Sakeperp class

``` 

from sakeperp import Sakeperp

address = "YOUR ADDRESS"         
private_key = "YOUR PRIVATE KEY"  
provider = "WEB3 PROVIDER URL"
test_mode = True
slippage = 0.005
sakePerp = Sakeperp(address=address, private_key=private_key, provider=provider, slippage=slippage, test_mode=test_mode)
 
``` 

#### Methods

#### wallet bnb balance
```
sakeperp.get_bnb_balance()
//84640780000000000
```

#### wallet busd balance
```
sakeperp.get_quote_balance()
//398709497939051062076623
```
    
#### Sakeperp opened Exchange List 
```
sakeperp.get_exchange_list()
//['ETH/BUSD', 'BTC/BUSD', 'BNB/BUSD', 'LINK/BUSD', 'DOT/BUSD', 'DOGE/BUSD', 'USDT/BUSD', 'EUR/BUSD', 'JPY/BUSD']
```

#### Open Position
```
exchange_name = "ETH/BUSD" 
side = Sakeperp.LONG_SIDE
quote_amount = int(10 * 1e18)
leverage = 1.5
check_waitperiod = False
sakeperp.open_position(exchange_name, side, quote_amount, leverage, check_waitperiod)
//0x8b2a495a5e9794859669c56bb803c3a7693b1b87a0b1ead5ffdd8c0dc7c9361e
```

#### Close Position
```
exchange_name = "ETH/BUSD" 
check_waitperiod = False
sakeperp.close_position(exchange_name, check_waitperiod)
```

#### Add Margin
```
exchange_name = "ETH/BUSD" 
margin = int(1 * 1e18)
sakeperp.add_margin(exchange_name, margin)
```

#### Remove Margin
```
exchange_name = "ETH/BUSD" 
margin = int(1 * 1e18)
sakeperp.remove_margin(exchange_name, margin)
```

#### Get Position On SakePerp
```
sakeperp.get_position("ETH/BUSD")
//{'size': -0.000760692639230579, 'margin': 2.0, 'open_notional': 2.0}
```

#### Get Position With Funding
```
sakeperp.get_position_with_funding("ETH/BUSD")
//{'size': -0.000760692639230579, 'margin': 2.0, 'open_notional': 2.0}
```

#### Get Funding And Overnigh Fee
```
sakeperp.get_funding_and_overnightfee("ETH/BUSD")
//{'fundingfee': 0.0, 'overnightfee': 0.0}
```
#### Marin Ratio
```
sakeperp.get_margin_ratio("ETH/BUSD")
//1.00001144994521
```
#### PositionNotional And Unrealizedpnl 
```
pnl_calc_optioin = Sakeperp.PNL_SPOT_PRICE
sakeperp.get_position_notional_and_unrealizedpnl("ETH/BUSD", pnl_calc_optioin)
//{'position_notional': 1.9999885501203405, 'unrealized_pnl': 1.1449879659593e-05}

```
#### Get Trading State
```
sakeperp.get_trading_state("ETH/BUSD")
//{'lastest_longtime': 0, 'lastest_shorttime': 1625210298}
```
#### Spot Price
```
sakeperp.get_spot_price("ETH/BUSD")
//2629.167626350927
```
#### Init Margin Ratio
```
sakeperp.get_init_margin_ratio("ETH/BUSD")
//0.1
```
#### Maintenance Ratio
```
sakeperp.get_maintenance_margin_ratio("ETH/BUSD")
//0.03
```
#### Spread Ratio
```
sakeperp.get_spread_ratio("ETH/BUSD")
//0.001
```

