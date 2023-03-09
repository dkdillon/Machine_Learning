# Machine_Learning
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import yfinance as yf
df = yf.download('AAPL', start='2019-01-01', end='2022-03-09')

# Compute moving averages
df['SMA5'] = df['Close'].rolling(window=5).mean()
df['SMA120'] = df['Close'].rolling(window=120).mean()

# Generate trading signals
df['Signal'] = 0.0
df['Signal'] = np.where(df['SMA5'] > df['SMA120'], 1.0, 0.0)
df['Position'] = df['Signal'].diff()

# Set initial capital
initial_capital = float(100000.0)

# Calculate daily returns
df['Returns'] = df['Close'].pct_change()

# Calculate positions and account value
df['Position'] = df['Position'].fillna(0.0)
df['Position'] = df['Position'].replace(-1.0, 0.0)
df['Position'] = df['Position'].replace(1.0, 1.0)
df['Position'] = df['Position'].replace(0.0, np.nan)
df['Position'] = df['Position'].fillna(method='ffill')
df['Position'] = df['Position'].fillna(0.0)
df['Account_Value'] = df['Position'] * df['Returns']

# Calculate cumulative returns
df['Account_Value'] = df['Account_Value'].cumsum() + initial_capital
df['Returns'] = df['Account_Value'].pct_change()

# Plot cumulative returns
df['Account_Value'].plot(figsize=(10,6))
plt.title('Cumulative Returns')

# Calculate performance metrics
total_returns = df['Account_Value'][-1] / df['Account_Value'][0] - 1.0
annualized_returns = ((1 + total_returns) ** (252/len(df)) - 1) * 100
sharpe_ratio = (df['Returns'].mean() / df['Returns'].std()) * np.sqrt(252)
max_drawdown = (df['Account_Value'] - df['Account_Value'].cummax()).min()

print('Total returns: {:.2f}%'.format(total_returns * 100))
print('Annualized returns: {:.2f}%'.format(annualized_returns))
print('Sharpe ratio: {:.2f}'.format(sharpe_ratio))
print('Max drawdown: {:.2f}%'.format(max_drawdown * 100))

# Calculate trading signal based on SMA crossover strategy
df['Signal'] = 0.0
df['Short_SMA'] = df['Close'].rolling(window=short_window, min_periods=1, center=False).mean()
df['Long_SMA'] = df['Close'].rolling(window=long_window, min_periods=1, center=False).mean()

df['Signal'][short_window:] = np.where(df['Short_SMA'][short_window:] > df['Long_SMA'][short_window:], 1.0, 0.0)

df['Position'] = df['Signal'].diff()

# Calculate returns based on trading signal
df['Returns'] = df['Close'].pct_change() * df['Position'].shift(1)

# Calculate cumulative returns
df['Cumulative_Returns'] = (1 + df['Returns']).cumprod()

import matplotlib.pyplot as plt

# Plot cumulative returns
plt.plot(df['Cumulative_Returns'])
plt.xlabel('Date')
plt.ylabel('Cumulative Returns')
plt.title('SMA Crossover Trading Strategy')
plt.show()
