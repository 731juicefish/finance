# finance

pip install pandas numpy matplotlib yfinance backtrader

import pandas as pd
import yfinance as yf

# 下載 TSM 股票數據（台積電）
tsm = yf.download('TSM', start='2023-01-01', end='2024-01-01')

# 顯示前 5 筆數據
print(tsm.head())

# 計算 20 天均線 和 50 天均線
tsm['SMA20'] = tsm['Close'].rolling(window=20).mean()
tsm['SMA50'] = tsm['Close'].rolling(window=50).mean()

# 顯示前 10 筆數據
print(tsm[['Close', 'SMA20', 'SMA50']].head(10))

# 計算每日報酬率
tsm['Returns'] = tsm['Close'].pct_change()

# 計算年化波動率
volatility = tsm['Returns'].std() * (252 ** 0.5)  # 252 表示一年交易日
print("年化波動率:", volatility)

# 計算累積最大值
tsm['Cumulative Max'] = tsm['Close'].cummax()

# 計算回撤
tsm['Drawdown'] = (tsm['Close'] - tsm['Cumulative Max']) / tsm['Cumulative Max']

# 找出最大回撤
max_drawdown = tsm['Drawdown'].min()
print("最大回撤:", max_drawdown)

import matplotlib.pyplot as plt

plt.figure(figsize=(12,6))
plt.plot(tsm.index, tsm['Close'], label='TSM 收盤價', color='black')
plt.plot(tsm.index, tsm['SMA20'], label='20日均線', color='blue', linestyle='dashed')
plt.plot(tsm.index, tsm['SMA50'], label='50日均線', color='red', linestyle='dashed')

plt.title('台積電 (TSM) 移動平均線策略')
plt.xlabel('日期')
plt.ylabel('價格')
plt.legend()
plt.show()

import backtrader as bt

# 建立策略
class SmaCross(bt.Strategy):
    params = (('sma1', 20), ('sma2', 50))  # 20 日均線 & 50 日均線

    def __init__(self):
        self.sma1 = bt.indicators.SimpleMovingAverage(self.data, period=self.params.sma1)
        self.sma2 = bt.indicators.SimpleMovingAverage(self.data, period=self.params.sma2)

    def next(self):
        if self.sma1[0] > self.sma2[0] and self.sma1[-1] <= self.sma2[-1]:  # 均線黃金交叉
            self.buy()
        elif self.sma1[0] < self.sma2[0] and self.sma1[-1] >= self.sma2[-1]:  # 均線死亡交叉
            self.sell()

# 設定回測環境
cerebro = bt.Cerebro()
data = bt.feeds.PandasData(dataname=tsm)
cerebro.adddata(data)
cerebro.addstrategy(SmaCross)

# 進行回測
cerebro.run()
cerebro.plot()
