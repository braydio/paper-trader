import backtrader as bt
import datetime

# Define a basic moving average crossover strategy
class SmaCrossover(bt.Strategy):
    params = (
        ('short_period', 10),
        ('long_period', 30),
    )

    def __init__(self):
        # Define short and long moving averages
        self.short_sma = bt.indicators.SimpleMovingAverage(self.data.close, period=self.params.short_period)
        self.long_sma = bt.indicators.SimpleMovingAverage(self.data.close, period=self.params.long_period)

    def next(self):
        # Check if we are in a position
        if not self.position:  # Not in the market
            if self.short_sma[0] > self.long_sma[0]:  # Buy signal
                self.buy()
                print(f"BUY executed at {self.data.close[0]} on {self.data.datetime.date(0)}")
        else:  # Already in the market
            if self.short_sma[0] < self.long_sma[0]:  # Sell signal
                self.close()
                print(f"SELL executed at {self.data.close[0]} on {self.data.datetime.date(0)}")

# Set up backtesting environment
cerebro = bt.Cerebro()
cerebro.addstrategy(SmaCrossover)

# Load historical data
data = bt.feeds.GenericCSVData(
    dataname='aapl_data.csv',  # Replace with your data file path
    dtformat='%Y-%m-%d',
    datetime=0,
    open=1,
    high=2,
    low=3,
    close=4,
    volume=5,
    openinterest=-1  # No open interest data
)

cerebro.adddata(data)

# Set initial cash and commission
cerebro.broker.set_cash(10000)  # Starting with $10,000
cerebro.broker.setcommission(commission=0.001)  # 0.1% commission per trade

# Run the backtest
print('Starting Portfolio Value:', cerebro.broker.getvalue())
cerebro.run()
print('Ending Portfolio Value:', cerebro.broker.getvalue())

# Plot the results
cerebro.plot()
