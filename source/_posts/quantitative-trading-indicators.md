---
title: 量化交易的常用指标
date: 2025-08-02 10:31:58
categories: 量化交易
tags:
  - 量化交易的指标
---
# 定义
- open：开盘价
- close： 收盘价
- high：最高价
- low：最低价
- volume：成交量

# MA
也即简单移动平均线(Sample Moving Average)，就是把最近N天的收盘价加起来然后除以N。
通常用在趋势分析上，比如5日均线、20日均线。
当5日均线上穿20日均线时，认为是买入信号；当5日均线下穿20日均线时，认为是卖出信号。
``` python
def _analysis_MA(df):
    df['MA5'] = ta.MA(df['close'], timeperiod=5)
    df['MA20'] = ta.MA(df['close'], timeperiod=20)
    # 买入信号：5日均线上穿20日均线（金叉）
    df['buy_signal'] = (df['MA5'] > df['MA20']) & (df['MA5'].shift(1) <= df['MA20'].shift(1))

    # 卖出信号：5日均线下穿20日均线（死叉）
    df['sell_signal'] = (df['MA5'] < df['MA20']) & (df['MA5'].shift(1) >= df['MA20'].shift(1))
```

# EMA
指数移动平均线（Exponential Moving Average，简称EMA），是以指数递减的方式计算得出的加权移动平均数。日期越近的权重越大。

``` python
def _analysis_EMA(df):
    df['EMA5'] = ta.EMA(df['close'], timeperiod=5)
    df['EMA20'] = ta.EMA(df['close'], timeperiod=20)
    # 买入信号：5日均线上穿20日均线（金叉）
    df['buy_signal'] = (df['EMA5'] > df['EMA20']) & (df['EMA5'].shift(1) <= df['EMA20'].shift(1))

    # 卖出信号：5日均线下穿20日均线（死叉）
    df['sell_signal'] = (df['EMA5'] < df['EMA20']) & (df['EMA5'].shift(1) >= df['EMA20'].shift(1))
```

# MACD
MACD（Moving Average Convergence Divergence）指标是利用短期（常用为12日EMA，即指数移动平均线）均线与长期（常用为26 日EMA）均线的聚合与分离状况，对买进、卖出时机作出研判的技术分析工具。

``` python
def _analysis_macd(df):
    # 计算MACD
    df['macd'], df['signal'], df['hist'] = ta.MACD(df['close'], 
                                                    fastperiod=12,  
                                                    slowperiod=26,  
                                                    signalperiod=9)
    # 买入信号：MACD线从下往上穿过信号线（金叉）
    df['buy_signal'] = (df['macd'] > df['signal']) & (df['macd'].shift(1) <= df['signal'].shift(1))

    # 卖出信号：MACD线从上往下穿过信号线（死叉）
    df['sell_signal'] = (df['macd'] < df['signal']) & (df['macd'].shift(1) >= df['signal'].shift(1))
```

# RSI
RSI（Relative Strength Index）指标，即相对强弱指数。计算是否是超买或者超卖。 
计算方式是计算一定时期内的收盘价涨幅的平均值与跌幅的平均值的比值，来衡量多空力量。
``` python
def _analysis_rsi(df):
    # 计算RSI 
    df['RSI'] = ta.RSI(df['close'], timeperiod=14)
    # 生成RSI买卖信号
    # 超卖区域(买入信号): RSI从下方穿过30
    df['buy_signal'] = (df['RSI'] > 30) & (df['RSI'].shift(1) <= 30)

    # 超买区域(卖出信号): RSI从上方穿过70
    df['sell_signal'] = (df['RSI'] < 70) & (df['RSI'].shift(1) >= 70)
```
# BOLL
BOLL（Bollinger Bands）指标，即布林带。具有通道的作用，可以反映出市场的波动情况。超过上轨，认为是超买；低于下轨，认为是超卖。
计算方式是计算一定时期内的收盘价的标准差和均值，然后在均值上加上或减去标准差的倍数。
``` python
def _analysis_boll(df):
    # 计算Boll
    df['upper'], df['middle'], df['lower'] = ta.BBANDS(df['close'], timeperiod=20,
                                                                nbdevup=2,
                                                                nbdevdn=2,
                                                                matype=0
                                                        )
    # 生成布林带买卖信号
    # 买入信号：价格从下向上突破下轨
    df['buy_signal'] = (df['close'] > df['lower']) & (df['close'].shift(1) <= df['lower'].shift(1))

    # 卖出信号：价格从上向下突破上轨
    df['sell_signal'] = (df['close'] < df['upper']) & (df['close'].shift(1) >= df['upper'].shift(1))
```

# CCI
CCI（Commodity Channel Index）指标，即商品通道指数。主要用于判断股价是否已超出常态分布范围。
计算方式是典型价格与移动平均的差除以平均绝对偏差。
``` python
def __analysis_cci(df, period=20):
    # 计算典型价格 (Typical Price)
    df['tp'] = (df['high'] + df['low'] + df['close']) / 3
    
    # 计算TP的N周期简单移动平均
    df['sma_tp'] = df['tp'].rolling(window=period).mean()
    
    # 计算平均绝对偏差 (Mean Deviation)
    df['mad'] = df['tp'].rolling(window=period).apply(lambda x: np.mean(np.abs(x - x.mean())))
    
    # 计算CCI，确保所有操作数都是float类型
    df['cci'] = (df['tp'].astype(float) - df['sma_tp'].astype(float)) / (0.015 * df['mad'].astype(float))
    
    # 处理可能的无穷值或NaN
    df['cci'] = df['cci'].replace([np.inf, -np.inf], np.nan).fillna(0)

    # 生成买卖信号
    # 买入信号：CCI从下方上穿-100
    df['buy_signal'] = (df['cci'] > -100) & (df['cci'].shift(1) <= -100)
    
    # 卖出信号：CCI从上方下穿+100
    df['sell_signal'] = (df['cci'] < 100) & (df['cci'].shift(1) >= 100)
  ```

# 总结
对于历史上波动率比较大的股票，可以考虑使用CCI指标。然后再考虑支撑位和阻力位。进行买入和卖出。
在财报或者重大事件也是很大的影响因素  