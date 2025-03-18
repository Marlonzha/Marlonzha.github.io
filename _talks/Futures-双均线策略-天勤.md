---
title: "双均线策略"
collection: talks
type: "Talk"
permalink: /talks/Futures-双均线策略-天勤
venue: "策略的测试和研究"
date: 2025-03-18
location: "China Taicang"
---

#天勤量化的双均线策略：https://doc.shinnytech.com/tqsdk/latest/demo/strategy.html#doublema  
##思路：  
定义长短周期：Long=60，Short=30，标的，通过计算周期内标的的收盘价的移动平均值：MA60和MA30的上下关系来作为开平仓依据。  
##这种均线策略：在大幅度、长时间的波动中能有不错的收益。在短期内的横盘震荡行情中会出现持续亏损。  
##回测下来发现问题：  
长短周期较短时，如MA15、MA5会出现交叉点不能买入的情况。原因是订阅的K线`klines["close"]`并不是总是返回每天的收盘价格。  
经过仔细核对发现`klines.iloc[-1]["close"]`是当日的开盘价格。 因为： 天勤内置的函数：api.is_changing：判断到参数内的指定字段变化后就会更新。导致只能获取到每天的开盘价和历史的收盘价格这样一个奇怪的组合。
/```python
def is_changing(self, obj: Any, key: Union[str, List[str], None] = None) -> bool:
        """
        判定obj最近是否有更新

        当业务数据更新导致 wait_update 返回后可以使用该函数判断 **本次业务数据更新是否包含特定obj或其中某个字段** 。

        关于判断K线更新的说明：
        当生成新K线时，其所有字段都算作有更新，若此时执行 api.is_changing(klines.iloc[-1]) 则一定返回True。

        Args:
            obj (any): 任意业务对象, 包括 get_quote 返回的 quote, get_kline_serial 返回的 k_serial, get_account 返回的 account 等

            key (str/list of str): [可选]需要判断的字段，默认不指定
                                  * 不指定: 当该obj下的任意字段有更新时返回True, 否则返回 False.
                                  * str: 当该obj下的指定字段有更新时返回True, 否则返回 False.
                                  * list of str: 当该obj下的指定字段中的任何一个字段有更新时返回True, 否则返回 False
/```

