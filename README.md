# BackTesting
import pandas as pd
import yfinance as yf
import datetime as dt
import matplotlib.pyplot as plt

start_date = dt.datetime.now() - dt.timedelta(days=365)
end_date = dt.datetime.now()

ticker = "FTSE"  # Replace with the ticker of the stock you want to backtest
data = yf.download(ticker, start=start_date, end=end_date)

data["Position"] = 0

for i in range(len(data)):
    current_date = data.index[i]
    previous_date = data.index[i - 1] if i > 0 else None

    # Buy on Monday at 11 am
    if current_date.weekday() == 0 and current_date.time() == dt.time(11, 0):
        data.at[current_date, "Position"] = 1

    # Sell on Tuesday at 11 am
    if previous_date and previous_date.weekday() == 0 and previous_date.time() == dt.time(11, 0):
        data.at[current_date, "Position"] = -1

data["Returns"] = data["Adj Close"].pct_change()
data["StrategyReturns"] = data["Position"].shift() * data["Returns"]

data["CumulativeReturns"] = (1 + data["StrategyReturns"]).cumprod()


plt.plot(data["CumulativeReturns"])
plt.title("Backtest Results - Buy Monday 11am, Sell Tuesday 11am")
plt.xlabel("Date")
plt.ylabel("Cumulative Returns")
plt.show()
